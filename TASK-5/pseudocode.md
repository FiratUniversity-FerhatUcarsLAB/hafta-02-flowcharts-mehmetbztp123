# ====== Sabitler / Türler ======
enum ThreatLevel { NONE, LOW, MEDIUM, HIGH, CRITICAL }

const LOOP_INTERVAL_MS = 500            # her döngü bekleme süresi
const SENSOR_DEBOUNCE_MS = 200         # aynı sensör için debounce
const FALSE_POSITIVE_THRESHOLD = 2     # kaç okumada tutarlıysa gerçek say
const NOTIFICATION_RETRY = 3
const ESCALATION_DELAY_MS = 30000      # yükseltme aralığı (örnek)

# ====== Global Durum ======
state alarmActive = false
state alarmLevel = NONE
state lastSensorTimestamps = map(sensorId -> 0)
state sensorStableCounts = map(sensorId -> 0)
state lastEscalationTime = 0

# ====== Fonksiyonlar ======
function ReadAllSensors():
    readings = map()
    for sensor in SensorList:
        readings[sensor.id] = sensor.read()        # örn: {motion:true, door:false, smoke:false}
    return readings

function NormalizeReadings(readings):
    # sensörleri debounce ve basit stabilite kontrolünden geçir
    stable = map()
    currentTime = Now()
    for (id, value) in readings:
        if currentTime - lastSensorTimestamps[id] < SENSOR_DEBOUNCE_MS:
            # debounce: önceki okunana yakın zamanda okunduysa sayma
            # burada basit davranış: ignore veya önceki ile karşılaştır
            pass
        lastSensorTimestamps[id] = currentTime

        if value == ExpectedValueForNoThreat(id): 
            sensorStableCounts[id] = 0
        else:
            sensorStableCounts[id] += 1

        stable[id] = (sensorStableCounts[id] >= FALSE_POSITIVE_THRESHOLD)
    return stable    # map sensorId -> boolean (gerçek/false positive filtresi)

function AssessThreatLevel(stableReadings):
    # basit kurallar: hangi sensör aktifse seviyeyi belirle
    level = NONE
    if stableReadings[SMOKE_SENSOR] or stableReadings[CO_SENSOR]:
        return CRITICAL
    if stableReadings[GLASS_BREAK_SENSOR] or stableReadings[DOOR_FORCED_SENSOR]:
        level = max(level, HIGH)
    if stableReadings[MOTION_SENSOR] and HomeMode == AWAY:
        level = max(level, MEDIUM)
    if stableReadings[MOTION_SENSOR] and HomeMode == HOME:
        level = max(level, LOW)
    # başka kurallar eklenebilir (kamera doğrulama, time-of-day, vs.)
    return level

function TriggerAlarm(level):
    alarmActive = true
    alarmLevel = level
    # donanımsal alarm birimini tetikle
    AlarmHardware.activate(level)    # seviye bazlı ton/volüm
    LogEvent("Alarm triggered", level, Now())

function MaintainAlarmUntilReset():
    # alarmActive true olduğu sürece alarm devam eder.
    while alarmActive:
        # sürekli siren / ışık / kayıt devam eder
        AlarmHardware.ensureActive(alarmLevel)
        # bildirimleri periyodik gönder (ör: her 30s)
        if timeToSendPeriodicNotification():
            SendNotification("ALARM_ACTIVE", alarmLevel)
        # kontrol: eğer kullanıcı/merkez sıfırlama yapmışsa alarmActive = false
        if ReceivedResetCommand():
            AlarmHardware.deactivate()
            LogEvent("Alarm reset by user/center", Now())
            alarmActive = false
            alarmLevel = NONE
            break
        Sleep(1000)   # kısa bekleme, döngüyü tıkama

function SendNotification(type, level):
    payload = BuildPayload(type, level)
    for attempt in 1..NOTIFICATION_RETRY:
        success = NotificationService.send(payload)
        if success: 
            LogEvent("Notification sent", type, level, Now())
            return true
        Sleep(1000 * attempt)   # geri çekilme (backoff)
    LogEvent("Notification failed", type, level, Now())
    return false

function LogEvent(message, ...args):
    # merkezi log sistemi / audit trail
    Logger.write({time:Now(), message:message, data:args})

function ReceivedResetCommand():
    # örneğin: kullanıcı uygulaması, fiziksel panel veya alarm merkezi komutu
    if UserInterface.hasPendingCommand("RESET_ALARM"):
        UserInterface.consumeCommand("RESET_ALARM")
        return true
    return false

# ====== Ana Döngü (7/24 çalışır) ======
DO
    startTime = Now()

    # 1) Tüm sensörleri oku
    rawReadings = ReadAllSensors()

    # 2) Okumaları normalize et / false positive filtresi uygula
    stableReadings = NormalizeReadings(rawReadings)

    # 3) Tehdit seviyesini değerlendir
    detectedLevel = AssessThreatLevel(stableReadings)

    # 4) Seviyeye göre aksiyon al
    if detectedLevel == NONE:
        # normal işletim: eğer alarm aktif değilse log yalnızca
        if alarmActive == false:
            LogEvent("No threat", Now())
        else:
            # eğer alarm halen aktifse, alarm dış tetikleme kontrollerini yap (ör: otomatik kapanma yok)
            LogEvent("Alarm still active, waiting for reset", alarmLevel, Now())
            # alarm sıfırlama ayrı fonksiyon içinde ele alınır (MaintainAlarmUntilReset)
    else
        # Eğer yeni veya yükselen bir tehdit varsa işlem başlat
        LogEvent("Threat detected", detectedLevel, Now())

        if alarmActive:
            # mevcut alarm varsa level'ı yükseltme mantığı
            if detectedLevel > alarmLevel:
                alarmLevel = detectedLevel
                AlarmHardware.updateLevel(alarmLevel)
                SendNotification("ALARM_ESCALATION", alarmLevel)
                LogEvent("Alarm escalated", alarmLevel, Now())
        else:
            # alarm aktif değilse seviyesi MEDIUM veya üzeriyse alarm başlat
            if detectedLevel >= MEDIUM:
                TriggerAlarm(detectedLevel)
                # başlangıç bildirimleri
                SendNotification("ALARM_TRIGGERED", detectedLevel)
                # alarm sıfırlanana kadar devam eden alt-döngü
                MaintainAlarmUntilReset()
            else
                # LOW tehdit: sadece bildirim / kamera kayıt başlat
                SendNotification("LOW_THREAT", detectedLevel)
                CameraSystem.recordSnapshotAll()
                # gerekirse izleme (escalation) için zaman damgası tut
                lastEscalationTime = Now()

    # 5) Eski ama çözülmemiş LOW/MEDIUM durumları için yükseltme kontrolü
    if alarmActive == false and detectedLevel >= LOW:
        if Now() - lastEscalationTime > ESCALATION_DELAY_MS and detectedLevel >= MEDIUM:
            # otomatik yükseltme örneği (opsiyonel)
            TriggerAlarm(detectedLevel)
            SendNotification("AUTO_ESCALATED_TO_ALARM", detectedLevel)
            MaintainAlarmUntilReset()

    # 6) Periyodik bakım ve günlük işlemler
    SystemHealth.checkAndReport()
    CleanupOldLogsIfNeeded()

    # 7) Döngü zamanlaması (sabit periyod)
    elapsed = Now() - startTime
    if elapsed < LOOP_INTERVAL_MS:
        Sleep(LOOP_INTERVAL_MS - elapsed)

WHILE (TRUE)

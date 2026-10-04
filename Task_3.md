# Практическая работа №3

# ЧАСТЬ I.

## Тема 1. Тег `<manifest>`

**1.1 Пространство имён tools**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.university.mobileapp">
</manifest>
```

**1.2 Удаление разрешения библиотеки**
```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" tools:node="remove" />
```

**1.3 Принудительная замена атрибута**
```xml
<application
    android:allowBackup="false"
    tools:replace="android:allowBackup" />
```

**1.4 sharedUserId** (устарел с API 29; нельзя добавить уже опубликованному приложению; оба приложения подписаны одним ключом)
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.university.mobileapp"
    android:sharedUserId="com.university.shared" />
```
Замена: ContentProvider / bound service с `signature`-разрешением.

**1.5 Минимальный валидный каркас**
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application android:label="@string/app_name" android:icon="@mipmap/ic_launcher">
        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

## Тема 2. Тег `<application>`

**2.1 Запрет бэкапа (финтех)**
```xml
<application
    android:allowBackup="false"
    android:fullBackupContent="false"
    android:dataExtractionRules="@xml/data_extraction_rules" />
```
`res/xml/data_extraction_rules.xml`:
```xml
<data-extraction-rules>
    <cloud-backup><exclude domain="root" /><exclude domain="database" />
        <exclude domain="sharedpref" /><exclude domain="file" /></cloud-backup>
    <device-transfer><exclude domain="root" /><exclude domain="database" />
        <exclude domain="sharedpref" /><exclude domain="file" /></device-transfer>
</data-extraction-rules>
```
`allowBackup="false"` блокирует и `adb backup`.

**2.2 SSL-pinning**
```xml
<application android:networkSecurityConfig="@xml/network_security_config" />
```
`res/xml/network_security_config.xml`:
```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors><certificates src="system" /></trust-anchors>
    </base-config>
    <domain-config>
        <domain includeSubdomains="true">api.university.ru</domain>
        <pin-set expiration="2027-12-31">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- обязательный резервный пин -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

**2.3 Кастомный Application**
```xml
<application android:name=".MainApplication" ... />
```
```java
public class MainApplication extends Application {
    @Override public void onCreate() {
        super.onCreate();
        // DI-контейнер, аналитика
    }
}
```

**2.4 Локализованное название**
```xml
<application android:label="@string/app_name" />
```
`values/strings.xml`: `<string name="app_name">My App</string>`
`values-ru/strings.xml`: `<string name="app_name">Моё приложение</string>`
Смена языка в рантайме: `AppCompatDelegate.setApplicationLocales(LocaleListCompat.forLanguageTags("ru"))`.

**2.5 largeHeap**
```xml
<application android:largeHeap="true" />
```
Риски: более долгие паузы GC; система агрессивнее убивает фоновые процессы; маскирует утечки памяти вместо их исправления; на разных устройствах лимит разный. Использовать только если профилирование доказало необходимость (большие bitmap в редакторе).

## Тема 3. Компоненты

**3.1 Единственная стартовая точка**
```xml
<activity android:name=".SplashScreenActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

**3.2 Изолированный сервис**
```xml
<service android:name=".sync.DbSyncService" android:exported="false" />
```

**3.3 Без пересоздания при повороте**
```xml
<activity android:name=".game.GameActivity" android:exported="false"
    android:configChanges="orientation|screenSize|keyboardHidden" />
```

**3.4 singleInstance**
```xml
<activity android:name=".call.IncomingCallActivity" android:exported="false"
    android:launchMode="singleInstance"
    android:showWhenLocked="true" android:turnScreenOn="true" />
```

**3.5 FileProvider для камеры**
```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
<queries><intent><action android:name="android.media.action.IMAGE_CAPTURE" /></intent></queries>
```
`res/xml/file_paths.xml`:
```xml
<paths><cache-path name="camera" path="images/" /></paths>
```
```java
Uri uri = FileProvider.getUriForFile(ctx, ctx.getPackageName() + ".fileprovider", photoFile);
intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
```

## Тема 4. Intent-фильтры

**4.1 Share Target (изображения)**
```xml
<activity android:name=".ShareReceiverActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

**4.2 Ссылки `tel:`**
```xml
<intent-filter>
    <action android:name="android.intent.action.DIAL" />
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="tel" />
</intent-filter>
```

**4.3 Только PDF по маске**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="file" /><data android:scheme="content" />
    <data android:host="*" />
    <data android:mimeType="*/*" />
    <data android:pathPattern=".*\\.pdf" />
</intent-filter>
```

**4.4 Несколько MIME-типов**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:mimeType="text/plain" />
    <data android:mimeType="text/html" />
</intent-filter>
```

**4.5 App Link с autoVerify**
```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="store.university.ru" />
</intent-filter>
```
`https://store.university.ru/.well-known/assetlinks.json`:
```json
[{ "relation": ["delegate_permission/common.handle_all_urls"],
   "target": { "namespace": "android_app",
               "package_name": "com.university.mobileapp",
               "sha256_cert_fingerprints": ["AA:BB:...:FF"] } }]
```
Назначение: система скачивает файл и проверяет, что домен «разрешает» именно это приложение (по пакету и отпечатку подписи) — ссылки открываются без диалога выбора и без угона чужим приложением.

## Тема 5. Разрешения

**5.1 Геотрекер**
```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
```
С Android 11 фоновую геолокацию запрашивают отдельным запросом после получения «в процессе использования».

**5.2 POST_NOTIFICATIONS**
```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```
На Android ≤ 12 разрешение игнорируется (уведомления разрешены по умолчанию), поэтому в коде запрос делается только при `Build.VERSION.SDK_INT >= 33`.

**5.3 Собственное signature-разрешение**
```xml
<permission android:name="com.university.permission.BIND_SYNC"
    android:protectionLevel="signature" />
<service android:name=".SyncService" android:exported="true"
    android:permission="com.university.permission.BIND_SYNC" />
```
Другое приложение с тем же ключом подписи объявляет `<uses-permission android:name="com.university.permission.BIND_SYNC" />`.

**5.4 maxSdkVersion**
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="28" />
```

**5.5 Точные будильники**
```xml
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
```
Google Play разрешает это только приложениям, у которых точное время — основная функция (будильник, таймер, календарь); нужна декларация в Play Console. Начиная с Android 14 для новых установок разрешение по умолчанию отозвано — проверяйте `AlarmManager.canScheduleExactAlarms()` и ведите пользователя в настройки. Для будильников/календарей есть `USE_EXACT_ALARM` (выдаётся автоматически, строгая модерация). Актуальные правила сверяйте в Play Console Help.

## Тема 6. `<uses-feature>` и `<queries>`

**6.1 Отпечаток — необязательный**
```xml
<uses-feature android:name="android.hardware.fingerprint" android:required="false" />
```

**6.2 NFC обязателен**
```xml
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc" android:required="true" />
```

**6.3 Яндекс Карты / 2ГИС**
```xml
<queries>
    <package android:name="ru.yandex.yandexmaps" />
    <package android:name="ru.dublgis.dgismobile" />
</queries>
```
(Идентификаторы пакетов сверьте в магазине приложений.)

**6.4 Телефония необязательна**
```xml
<uses-feature android:name="android.hardware.telephony" android:required="false" />
<uses-feature android:name="android.hardware.telephony.messaging" android:required="false" />
```

**6.5 Геймпад (Android TV)**
```xml
<uses-feature android:name="android.software.leanback" android:required="true" />
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />
<uses-feature android:name="android.hardware.gamepad" android:required="false" />
```

## Тема 7. `<meta-data>`

**7.1 Яндекс MapKit** ⚠️ MapKit штатно получает ключ в коде, а не из манифеста. Манифест используем как хранилище (подстановка из Gradle):
```xml
<meta-data android:name="com.yandex.mapkit.API_KEY" android:value="${YANDEX_MAPKIT_KEY}" />
```
```java
// Application.onCreate(), до setContentView с MapView
String key = getPackageManager()
    .getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA)
    .metaData.getString("com.yandex.mapkit.API_KEY");
MapKitFactory.setApiKey(key);
```

**7.2 LocaleConfig** ⚠️ В условии указан `<meta-data>`; правильный способ — атрибут `<application>`:
```xml
<application android:localeConfig="@xml/locales_config" />
```
`res/xml/locales_config.xml`:
```xml
<locale-config xmlns:android="http://schemas.android.com/apk/res/android">
    <locale android:name="ru" /><locale android:name="en" />
</locale-config>
```

**7.3 searchable в Activity**
```xml
<activity android:name=".SearchActivity" android:exported="true">
    <intent-filter><action android:name="android.intent.action.SEARCH" /></intent-filter>
    <meta-data android:name="android.app.searchable" android:resource="@xml/searchable" />
</activity>
```
`res/xml/searchable.xml`:
```xml
<searchable xmlns:android="http://schemas.android.com/apk/res/android"
    android:label="@string/app_name" android:hint="@string/search_hint" />
```

**7.4 Ручная инициализация androidx.startup**
```xml
<provider android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false" tools:node="merge">
    <!-- отключить конкретный инициализатор -->
    <meta-data android:name="com.example.MyInitializer" tools:node="remove" />
</provider>
```
Полностью отключить: `<provider ... tools:node="remove" />`; затем `AppInitializer.getInstance(ctx).initializeComponent(MyInitializer.class)` в нужный момент.

**7.5 120 Гц** ⚠️ Стандартного ключа в манифесте для высокой частоты нет — это делается в коде:
```java
// Android 11+
getWindow().getAttributes().preferredRefreshRate = 120f;   // или
surface.setFrameRate(120f, Surface.FRAME_RATE_COMPATIBILITY_DEFAULT);
```
Любой `<meta-data>` для этого — вендорный и не гарантируется.

---

# ЧАСТЬ II. 50 практических заданий

## Блок 1. Архитектура и запуск

**1. overrideLibrary**
```xml
<uses-sdk tools:overrideLibrary="com.thirdparty.lib" />
```
Код библиотеки на API < 24 защищайте проверкой `Build.VERSION.SDK_INT`.

**2. Activity без иконки в лаунчере** (нет `LAUNCHER`)
```xml
<activity android:name=".PartnerEntryActivity" android:exported="true">
    <intent-filter>
        <action android:name="com.university.action.OPEN_PARTNER" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

**3. Splash Screen API**
```xml
<activity android:name=".MainActivity" android:exported="true"
    android:theme="@style/Theme.App.Starting">…</activity>
```
```xml
<style name="Theme.App.Starting" parent="Theme.SplashScreen">
    <item name="windowSplashScreenBackground">@color/brand</item>
    <item name="windowSplashScreenAnimatedIcon">@drawable/ic_splash</item>
    <item name="postSplashScreenTheme">@style/Theme.ModernApp</item>
</style>
```
В коде до `super.onCreate()`: `SplashScreen.installSplashScreen(this);` (зависимость `androidx.core:core-splashscreen`).

**4. activity-alias (смена иконки)**
```xml
<activity-alias android:name=".LauncherDefault" android:targetActivity=".MainActivity"
    android:enabled="true" android:exported="true" android:icon="@mipmap/ic_launcher">
    <intent-filter><action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" /></intent-filter>
</activity-alias>
<activity-alias android:name=".LauncherNewYear" android:targetActivity=".MainActivity"
    android:enabled="false" android:exported="true" android:icon="@mipmap/ic_launcher_ny">
    <intent-filter><action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" /></intent-filter>
</activity-alias>
```
```java
PackageManager pm = getPackageManager();
pm.setComponentEnabledSetting(new ComponentName(this, ".LauncherDefault"),
    PackageManager.COMPONENT_ENABLED_STATE_DISABLED, PackageManager.DONT_KILL_APP);
pm.setComponentEnabledSetting(new ComponentName(this, ".LauncherNewYear"),
    PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP);
```

**5. Picture-in-Picture**
```xml
<activity android:name=".VideoActivity" android:exported="false"
    android:supportsPictureInPicture="true" android:resizeableActivity="true"
    android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation" />
```

**6. Минимальный размер окна**
```xml
<activity android:name=".MainActivity" android:resizeableActivity="true" android:exported="true">
    <layout android:defaultWidth="600dp" android:defaultHeight="800dp"
        android:minWidth="300dp" android:minHeight="450dp" />
</activity>
```

**7. Отдельный процесс**
```xml
<service android:name=".playback.AudioService" android:process=":playback_process"
    android:exported="false" />
```
(`:` — приватный процесс приложения; `Application.onCreate` вызывается в каждом процессе.)

**8. Исключение из Recents**
```xml
<activity android:name=".MasterPasswordActivity" android:exported="false"
    android:excludeFromRecents="true" />
```

**9. Превью в Recents и скриншоты.** В манифесте отдельного атрибута для скрытия превью нет. `FLAG_SECURE` (в коде: `getWindow().setFlags(FLAG_SECURE, FLAG_SECURE)`) запрещает скриншоты, запись экрана и показывает пустое превью в Recents. `android:excludeFromRecents="true"` убирает задачу из списка целиком. С Android 13 есть `Activity.setRecentsScreenshotEnabled(false)` — прячет только превью.

**10. instrumentation**
```xml
<instrumentation android:name="com.university.mobileapp.CustomTestRunner"
    android:targetPackage="com.university.mobileapp" android:label="UI tests" />
```
Обычно это делается в Gradle: `testInstrumentationRunner "com.university.mobileapp.CustomTestRunner"`.

## Блок 2. Безопасность

**11. isolatedProcess**
```xml
<service android:name=".sandbox.ParserService" android:isolatedProcess="true"
    android:exported="false" />
```
Изолированный процесс не имеет разрешений и доступа к сети и ограничивает ущерб от взлома парсера. Сам по себе он не запрещает динамическую загрузку кода; защита — не грузить `dex` из записываемых мест (на Android 14+ такие файлы должны быть read-only).

**12. HTTP только для 192.168.1.50**
```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="false">192.168.1.50</domain>
    </domain-config>
</network-security-config>
```
Размещайте в `src/debug/res/xml`, чтобы в релиз не попало.

**13. Провайдер с раздельными правами**
```xml
<permission android:name="com.university.permission.READ_DATA" android:protectionLevel="signature" />
<permission android:name="com.university.permission.WRITE_DATA" android:protectionLevel="signature" />
<provider android:name=".data.NotesProvider" android:authorities="${applicationId}.notes"
    android:exported="true"
    android:readPermission="com.university.permission.READ_DATA"
    android:writePermission="com.university.permission.WRITE_DATA" />
```
Против SQL-инъекций — только параметризованные запросы (`selectionArgs`).

**14. grant-uri-permission**
```xml
<provider android:name=".data.DocsProvider" android:authorities="${applicationId}.docs"
    android:exported="false" android:grantUriPermissions="false">
    <grant-uri-permission android:pathPrefix="/shared_docs/" />
</provider>
```
Либо `grantUriPermissions="true"` (любые URI), либо вложенные `<grant-uri-permission>` — одновременно использовать нельзя.

**15. debuggable=false**
```xml
<application android:debuggable="false" tools:replace="android:debuggable" />
```
Лучше: `buildTypes { release { debuggable false } }` — сборщик сам выставит значение (жёстко прописанный атрибут вызывает lint-предупреждение).

**16. Защита ресивера**
```xml
<permission android:name="com.university.permission.INTERNAL_BROADCAST"
    android:protectionLevel="signature" />
<receiver android:name=".InternalReceiver" android:exported="false" />
<!-- если нужен приём от других своих приложений: -->
<receiver android:name=".SharedReceiver" android:exported="true"
    android:permission="com.university.permission.INTERNAL_BROADCAST" />
```

**17. BIND_ACCESSIBILITY_SERVICE**
```xml
<activity android:name=".A11ySettingsActivity" android:exported="true"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE" />
```
Это разрешение уровня signature/system: вызвать такую Activity сможет только система. Обычно им защищают `<service>` (см. №38).

**18. Backup rules**
```xml
<application android:allowBackup="true"
    android:fullBackupContent="@xml/backup_rules"
    android:dataExtractionRules="@xml/data_extraction_rules" />
```
`xml/backup_rules.xml` (Android ≤ 11):
```xml
<full-backup-content>
    <exclude domain="sharedpref" path="secure_prefs.xml" />
    <exclude domain="database" path="tokens.db" />
</full-backup-content>
```
`xml/data_extraction_rules.xml` (Android 12+):
```xml
<data-extraction-rules>
    <cloud-backup>
        <exclude domain="sharedpref" path="secure_prefs.xml" />
        <exclude domain="database" path="tokens.db" />
    </cloud-backup>
    <device-transfer><exclude domain="database" path="tokens.db" /></device-transfer>
</data-extraction-rules>
```

**19. Tapjacking.** Защита задаётся не в манифесте, а во View:
```xml
<Button android:id="@+id/confirm" android:filterTouchesWhenObscured="true" ... />
```
```java
confirmButton.setFilterTouchesWhenObscured(true);
```
Дополнительно (Android 12+): `getWindow().setHideOverlayWindows(true)` на экране подтверждения (нужно разрешение `HIDE_OVERLAY_WINDOWS`).

**20. Аудит манифеста** (фрагмента в условии нет — типовой уязвимый пример):
```xml
<application android:allowBackup="true" android:debuggable="true"
    android:usesCleartextTraffic="true">
    <activity android:name=".AdminActivity" android:exported="true" />
    <service android:name=".TokenService" android:exported="true" />
    <provider android:name=".UserProvider" android:authorities="x" android:exported="true" />
</application>
```
Уязвимости: (1) `exported="true"` у Activity/Service/Provider без `android:permission` — любое приложение вызовет компонент или прочитает данные; (2) `debuggable="true"` + `allowBackup="true"` — доступ к данным через `run-as`/`adb backup`; (3) `usesCleartextTraffic="true"` — перехват трафика (MITM). Исправление: `exported="false"`, `permission` signature, `debuggable="false"`, `allowBackup="false"`, `usesCleartextTraffic="false"`.

## Блок 3. Intent-фильтры

**21. NFC TECH_DISCOVERED**
```xml
<uses-permission android:name="android.permission.NFC" />
<activity android:name=".NfcActivity" android:exported="true">
    <intent-filter><action android:name="android.nfc.action.TECH_DISCOVERED" /></intent-filter>
    <meta-data android:name="android.nfc.action.TECH_DISCOVERED"
        android:resource="@xml/nfc_tech_filter" />
</activity>
```
`xml/nfc_tech_filter.xml`:
```xml
<resources>
    <tech-list><tech>android.nfc.tech.MifareClassic</tech></tech-list>
</resources>
```

**22. geo:**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="geo" />
</intent-filter>
```
Фильтр сопоставляется по схеме; параметр `?q=` разбирается в коде: `intent.getData().getSchemeSpecificPart()`.

**23. docx / xlsx**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:scheme="content" /><data android:scheme="file" />
    <data android:mimeType="application/*" />
</intent-filter>
```
Точнее: `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (docx) и `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (xlsx).

**24. Shortcuts / App Actions**
```xml
<activity android:name=".MainActivity" android:exported="true">
    <intent-filter>…LAUNCHER…</intent-filter>
    <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts" />
</activity>
```
`xml/shortcuts.xml`:
```xml
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut android:shortcutId="orders" android:enabled="true"
        android:icon="@drawable/ic_orders"
        android:shortcutShortLabel="@string/orders_short"
        android:shortcutLongLabel="@string/orders_long">
        <intent android:action="android.intent.action.VIEW"
            android:targetPackage="com.university.mobileapp"
            android:targetClass="com.university.mobileapp.ui.OrdersActivity" />
    </shortcut>
</shortcuts>
```
Для App Actions в этот же файл добавляют `<capability android:name="actions.intent.…">`.

**25. Deep Link магазина**
```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="marketplace.com"
        android:pathPrefix="/catalog" />
</intent-filter>
```
```java
String brand = getIntent().getData().getQueryParameter("brand"); // "nike"
```

**26. MEDIA_BUTTON**
```xml
<receiver android:name=".MediaButtonReceiver" android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>
```
Современный путь — `MediaSession` (`androidx.media.session.MediaButtonReceiver`).

**27. SEARCH**
```xml
<activity android:name=".SearchActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.SEARCH" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <meta-data android:name="android.app.searchable" android:resource="@xml/searchable" />
</activity>
```

**28. smsto:**
```xml
<intent-filter>
    <action android:name="android.intent.action.SENDTO" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="smsto" /><data android:scheme="sms" />
</intent-filter>
```

**29. OAuth-callback**
```xml
<activity android:name=".OAuthRedirectActivity" android:exported="true"
    android:launchMode="singleTop">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="org.example.app" android:host="oauth-callback" />
    </intent-filter>
</activity>
```
Используйте PKCE; для надёжности — App Link вместо кастомной схемы.

**30. Файлы `.mycfg`**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="file" /><data android:scheme="content" />
    <data android:host="*" />
    <data android:mimeType="*/*" />
    <data android:pathPattern=".*\\.mycfg" />
</intent-filter>
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:scheme="content" android:mimeType="application/x-mycfg" />
</intent-filter>
```
Для `content://` без расширения в URI срабатывает только фильтр по своему MIME-типу.

## Блок 4. Сервисы (Android 14+)

**31. Микрофон**
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MICROPHONE" />
<service android:name=".record.VoiceRecorderService" android:exported="false"
    android:foregroundServiceType="microphone" />
```

**32. Геолокация (курьер)**
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
<service android:name=".courier.TrackingService" android:exported="false"
    android:foregroundServiceType="location" />
```

**33. dataSync с таймаутом**
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
<service android:name=".DownloadService" android:exported="false"
    android:foregroundServiceType="dataSync" />
```
Начиная с Android 15 у `dataSync` есть лимит ≈6 часов в сутки; система вызывает `Service.onTimeout(int, int)` — в нём нужно остановить сервис (`stopSelf()`). Для долгих загрузок лучше user-initiated data transfer job (`RUN_USER_INITIATED_JOBS`) или WorkManager.

**34. mediaProjection**
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PROJECTION" />
<service android:name=".CastService" android:exported="false"
    android:foregroundServiceType="mediaProjection" />
```
Сервис запускается только после получения согласия пользователя (`createScreenCaptureIntent`).

**35. WorkManager без авто-инициализации**
```xml
<provider android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false" tools:node="merge">
    <meta-data android:name="androidx.work.WorkManagerInitializer"
        android:value="androidx.startup" tools:node="remove" />
</provider>
```
```java
public class App extends Application implements Configuration.Provider {
    @NonNull @Override public Configuration getWorkManagerConfiguration() {
        return new Configuration.Builder().setMinimumLoggingLevel(Log.INFO).build();
    }
}
```

**36. Автозапуск после перезагрузки**
```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<receiver android:name=".BootReceiver" android:exported="false">
    <intent-filter><action android:name="android.intent.action.BOOT_COMPLETED" /></intent-filter>
</receiver>
```
```java
public void onReceive(Context c, Intent i) {
    if (Intent.ACTION_BOOT_COMPLETED.equals(i.getAction()))
        WorkManager.getInstance(c).enqueue(new OneTimeWorkRequest.Builder(SyncWorker.class).build());
}
```
С Android 15 из `BOOT_COMPLETED` нельзя стартовать ряд FGS (dataSync, mediaPlayback, microphone, camera и др.) — планируйте через WorkManager.

**37. Игнорирование оптимизации батареи**
```xml
<!-- Google Play: разрешено только если базовая функция приложения нарушается
     оптимизацией батареи (мессенджер, VoIP, трекер устройств). Иначе — отклонение. -->
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />
```

**38. AccessibilityService**
```xml
<service android:name=".access.MyAccessibilityService" android:exported="true"
    android:label="@string/a11y_label"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
    </intent-filter>
    <meta-data android:name="android.accessibilityservice"
        android:resource="@xml/accessibility_service_config" />
</service>
```
`xml/accessibility_service_config.xml`:
```xml
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowStateChanged"
    android:accessibilityFeedbackType="feedbackSpoken"
    android:notificationTimeout="100"
    android:description="@string/a11y_description" />
```
Google Play требует, чтобы такое приложение действительно помогало людям с ограниченными возможностями.

**39. SyncAdapter**
```xml
<service android:name=".sync.SyncService" android:exported="true">
    <intent-filter><action android:name="android.content.SyncAdapter" /></intent-filter>
    <meta-data android:name="android.content.SyncAdapter" android:resource="@xml/syncadapter" />
</service>
```
`xml/syncadapter.xml`:
```xml
<sync-adapter xmlns:android="http://schemas.android.com/apk/res/android"
    android:contentAuthority="com.university.mobileapp.provider"
    android:accountType="com.university.account"
    android:userVisible="true" android:supportsUploading="true"
    android:allowParallelSyncs="false" />
```
(Нужны ещё `AccountAuthenticator`-сервис и провайдер. Для нового кода предпочтителен WorkManager.)

**40. Live Wallpaper**
```xml
<uses-feature android:name="android.software.live_wallpaper" android:required="true" />
<service android:name=".wallpaper.MyWallpaperService" android:exported="true"
    android:permission="android.permission.BIND_WALLPAPER">
    <intent-filter><action android:name="android.service.wallpaper.WallpaperService" /></intent-filter>
    <meta-data android:name="android.service.wallpaper" android:resource="@xml/wallpaper" />
</service>
```
`xml/wallpaper.xml`: `<wallpaper xmlns:android="…" android:thumbnail="@drawable/thumb" android:description="@string/wp_desc" />`

## Блок 5. Железо и адаптивность

**41. Складные устройства**
```xml
<activity android:name=".MainActivity" android:exported="true"
    android:resizeableActivity="true"
    android:configChanges="orientation|screenSize|smallestScreenSize|screenLayout|keyboardHidden"
    android:minAspectRatio="1.0" android:maxAspectRatio="2.4" />
```
Рекомендуется не ограничивать соотношение сторон вообще и реагировать на изменения через Jetpack WindowManager (`WindowInfoTracker`). `minAspectRatio` — API 31+.

**42. Wear OS**
```xml
<uses-feature android:name="android.hardware.type.watch" />
<application>
    <meta-data android:name="com.google.android.wearable.standalone" android:value="true" />
</application>
```

**43. Android Auto**
```xml
<application>
    <meta-data android:name="com.google.android.gms.car.application"
        android:resource="@xml/automotive_app_desc" />
</application>
```
`xml/automotive_app_desc.xml`:
```xml
<automotiveApp><uses name="media" /></automotiveApp>
```
(плюс `MediaBrowserService` с action `android.media.browse.MediaBrowserService`.)

**44. Android TV**
```xml
<uses-feature android:name="android.software.leanback" android:required="true" />
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />
<application android:banner="@drawable/tv_banner">
    <activity android:name=".tv.TvMainActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

**45. Стилус / расширенный сенсор**
```xml
<uses-feature android:name="android.hardware.touchscreen" android:required="true" />
<uses-feature android:name="android.hardware.touchscreen.multitouch.distinct" android:required="true" />
```
Отдельного флага «стилус» в `uses-feature` нет: тип ввода определяется в коде (`MotionEvent.TOOL_TYPE_STYLUS`).

**46. supports-screens** ⚠️ Этот тег управляет размером экрана, а не плотностью. Плотность ограничивает только `<compatible-screens>` (не рекомендуется) или каталог устройств Play.
```xml
<supports-screens android:smallScreens="false" android:normalScreens="true"
    android:largeScreens="true" android:xlargeScreens="true"
    android:requiresSmallestWidthDp="320" />
```

**47. Высокая частота сенсоров**
```xml
<uses-permission android:name="android.permission.HIGH_SAMPLING_RATE_SENSORS" />
```
(Normal-разрешение, Android 12+; нужно для опроса быстрее 200 Гц.)

**48. Вспышка — опционально**
```xml
<uses-feature android:name="android.hardware.camera.flash" android:required="false" />
```

**49. USB-аксессуары (OTG)**
```xml
<uses-feature android:name="android.hardware.usb.host" android:required="true" />
<activity android:name=".UsbActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED" />
    </intent-filter>
    <meta-data android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED"
        android:resource="@xml/device_filter" />
</activity>
```
`xml/device_filter.xml`:
```xml
<resources><usb-device vendor-id="1234" product-id="5678" /></resources>
```

**50. Production-манифест финтех-приложения**
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Normal -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.USE_BIOMETRIC" />
    <!-- Dangerous (3 шт.) -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <uses-feature android:name="android.hardware.camera" android:required="false" />
    <uses-feature android:name="android.hardware.location.gps" android:required="false" />

    <queries>
        <intent><action android:name="android.intent.action.VIEW" />
            <data android:scheme="https" /></intent>
    </queries>

    <application
        android:name=".FintechApp"
        android:allowBackup="false"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="false"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Fintech"
        android:usesCleartextTraffic="false"
        android:networkSecurityConfig="@xml/network_security_config"
        android:localeConfig="@xml/locales_config">

        <!-- Стартовый экран + App Link -->
        <activity android:name=".ui.SplashActivity" android:exported="true"
            android:theme="@style/Theme.App.Starting" android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="https" android:host="pay.university.ru"
                    android:pathPrefix="/pay" />
            </intent-filter>
        </activity>

        <!-- Экран подтверждения платежа: не в Recents, FLAG_SECURE + filterTouches в коде -->
        <activity android:name=".ui.ConfirmPaymentActivity" android:exported="false"
            android:excludeFromRecents="true" android:windowSoftInputMode="adjustResize" />

        <provider android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false" android:grantUriPermissions="true">
            <meta-data android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>
</manifest>
```
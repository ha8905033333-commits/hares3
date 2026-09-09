# hares3from pathlib import Path
import textwrap, zipfile, shutil, os

root = Path("/mnt/data/Hares")
if root.exists():
    shutil.rmtree(root)

files = {
"settings.gradle.kts": r'''
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "Hares"
include(":app")
''',
"build.gradle.kts": r'''
plugins {
    id("com.android.application") version "8.7.3" apply false
    id("org.jetbrains.kotlin.android") version "2.0.21" apply false
}
''',
"gradle.properties": r'''
org.gradle.jvmargs=-Xmx2g -Dfile.encoding=UTF-8
android.useAndroidX=true
kotlin.code.style=official
''',
"app/build.gradle.kts": r'''
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.hares.monitor"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.hares.monitor"
        minSdk = 26
        targetSdk = 36
        versionCode = 1
        versionName = "1.0.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            applicationIdSuffix = ".debug"
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    val camerax = "1.4.2"
    implementation("androidx.core:core-ktx:1.15.0")
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
    implementation("androidx.activity:activity-ktx:1.10.1")
    implementation("androidx.lifecycle:lifecycle-service:2.8.7")
    implementation("androidx.camera:camera-core:$camerax")
    implementation("androidx.camera:camera-camera2:$camerax")
    implementation("androidx.camera:camera-lifecycle:$camerax")
    implementation("androidx.camera:camera-view:$camerax")
}
''',
"app/proguard-rules.pro": r'''
# Hares currently ships without code shrinking.
''',
"app/src/main/AndroidManifest.xml": r'''
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_CAMERA" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />

    <application
        android:allowBackup="false"
        android:label="حارس"
        android:icon="@drawable/ic_hare"
        android:supportsRtl="true"
        android:theme="@style/Theme.Hares">

        <activity
            android:name=".SettingsActivity"
            android:exported="false" />

        <activity
            android:name=".GalleryActivity"
            android:exported="false" />

        <activity
            android:name=".PinActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".MainActivity"
            android:exported="false" />

        <service
            android:name=".MonitoringService"
            android:exported="false"
            android:foregroundServiceType="camera" />
    </application>
</manifest>
''',
"app/src/main/res/values/strings.xml": r'''
<resources>
    <string name="app_name">حارس</string>
    <string name="start_monitoring">بدء المراقبة</string>
    <string name="stop_monitoring">إيقاف المراقبة</string>
    <string name="gallery">المعرض</string>
    <string name="settings">الإعدادات</string>
</resources>
''',
"app/src/main/res/values/styles.xml": r'''
<resources>
    <style name="Theme.Hares" parent="Theme.Material3.DayNight.NoActionBar">
        <item name="android:fontFamily">sans</item>
        <item name="android:windowLightStatusBar">false</item>
        <item name="android:statusBarColor">#111111</item>
        <item name="android:navigationBarColor">#111111</item>
    </style>
</resources>
''',
"app/src/main/res/drawable/ic_hare.xml": r'''
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="48dp" android:height="48dp" android:viewportWidth="48" android:viewportHeight="48">
    <path android:fillColor="#FFFFFF"
        android:pathData="M10,30 C8,23 11,17 17,15 C15,10 16,6 19,5 C23,5 24,10 23,14 C26,13 29,13 31,14 C30,9 32,5 35,5 C39,6 38,12 36,16 C41,19 43,25 41,30 C39,37 31,40 24,39 C17,39 12,36 10,30z" />
</vector>
''',
"app/src/main/res/layout/activity_pin.xml": r'''
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:gravity="center" android:orientation="vertical" android:padding="28dp"
    android:background="#111111">
    <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:text="🛡️ حارس" android:textSize="34sp" android:textColor="#FFFFFF"
        android:textStyle="bold" />
    <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:text="مراقب الحركة" android:textSize="18sp" android:textColor="#BBBBBB"
        android:layout_marginTop="6dp" />
    <TextView android:id="@+id/pinTitle" android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:text="أنشئ رمز PIN من 6 أرقام" android:textColor="#FFFFFF" android:layout_marginTop="40dp" />
    <EditText android:id="@+id/pinInput" android:layout_width="240dp" android:layout_height="56dp"
        android:inputType="numberPassword" android:maxLength="6" android:gravity="center"
        android:textSize="24sp" android:layout_marginTop="14dp" />
    <Button android:id="@+id/pinButton" android:layout_width="240dp" android:layout_height="wrap_content"
        android:text="متابعة" android:layout_marginTop="16dp" />
    <TextView android:id="@+id/pinError" android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:textColor="#FF7777" android:layout_marginTop="10dp" />
</LinearLayout>
''',
"app/src/main/res/layout/activity_main.xml": r'''
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent" android:background="#000000">

    <androidx.camera.view.PreviewView
        android:id="@+id/preview"
        android:layout_width="match_parent" android:layout_height="match_parent" />

    <com.hares.monitor.OverlayView
        android:id="@+id/overlay"
        android:layout_width="match_parent" android:layout_height="match_parent" />

    <LinearLayout
        android:layout_width="match_parent" android:layout_height="wrap_content"
        android:layout_gravity="top" android:orientation="horizontal"
        android:padding="12dp" android:background="#66000000">
        <TextView android:id="@+id/status" android:layout_width="0dp" android:layout_height="wrap_content"
            android:layout_weight="1" android:text="🔴 متوقفة" android:textColor="#FFFFFF"
            android:textSize="16sp" />
        <Button android:id="@+id/cameraSwitch" android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="تبديل الكاميرا" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent" android:layout_height="wrap_content"
        android:layout_gravity="bottom" android:orientation="horizontal"
        android:gravity="center" android:padding="12dp" android:background="#88000000">
        <Button android:id="@+id/startStop" android:layout_width="0dp" android:layout_height="wrap_content"
            android:layout_weight="1" android:text="▶ بدء المراقبة" />
        <Button android:id="@+id/galleryButton" android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="🖼️ المعرض" android:layout_marginStart="8dp" />
        <Button android:id="@+id/settingsButton" android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="⚙️" android:layout_marginStart="8dp" />
    </LinearLayout>
</FrameLayout>
''',
"app/src/main/res/layout/activity_settings.xml": r'''
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">
    <LinearLayout android:layout_width="match_parent" android:layout_height="wrap_content"
        android:orientation="vertical" android:padding="20dp">
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="إعدادات حارس" android:textSize="28sp" android:textStyle="bold" />
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="كشف الحركة" android:textSize="20sp" android:layout_marginTop="24dp" />
        <TextView android:id="@+id/sensitivityLabel" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="الحساسية: 60" />
        <SeekBar android:id="@+id/sensitivity" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:max="100" android:progress="60" />
        <TextView android:id="@+id/minAreaLabel" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="الحد الأدنى للحركة: 80" />
        <SeekBar android:id="@+id/minArea" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:max="500" android:progress="80" />
        <TextView android:id="@+id/cooldownLabel" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="فترة التهدئة: 4 ثوانٍ" />
        <SeekBar android:id="@+id/cooldown" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:max="60" android:progress="4" />

        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="الأداء" android:textSize="20sp" android:layout_marginTop="24dp" />
        <Spinner android:id="@+id/performance" android:layout_width="match_parent" android:layout_height="wrap_content" />

        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="الإنذارات" android:textSize="20sp" android:layout_marginTop="24dp" />
        <Switch android:id="@+id/soundEnabled" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="🔊 الصوت" android:checked="true" />
        <SeekBar android:id="@+id/volume" android:layout_width="match_parent" android:layout_height="wrap_content" android:max="100" android:progress="80" />
        <Switch android:id="@+id/screenFlash" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="💡 وميض الشاشة" android:checked="true" />
        <Switch android:id="@+id/notificationEnabled" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="🔔 الإشعار" android:checked="true" />
        <Switch android:id="@+id/saveImages" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="📸 حفظ الصور" android:checked="true" />

        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="الشاشة والطاقة" android:textSize="20sp" android:layout_marginTop="24dp" />
        <Switch android:id="@+id/dimScreen" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="🌙 الشاشة الخافتة" />
        <Switch android:id="@+id/chargeOnly" android:layout_width="match_parent" android:layout_height="wrap_content" android:text="🔌 المراقبة أثناء الشحن فقط" />
        <Button android:id="@+id/batteryButton" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="🔋 السماح للتطبيق بتجاوز تحسين البطارية" />
        <Button android:id="@+id/changePin" android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="🔐 تغيير رمز PIN" />
    </LinearLayout>
</ScrollView>
''',
"app/src/main/res/layout/activity_gallery.xml": r'''
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:orientation="vertical" android:padding="12dp">
    <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:text="🖼️ معرض حارس" android:textSize="26sp" android:textStyle="bold" />
    <GridView android:id="@+id/grid" android:layout_width="match_parent" android:layout_height="0dp"
        android:layout_weight="1" android:numColumns="3" android:verticalSpacing="6dp"
        android:horizontalSpacing="6dp" android:stretchMode="columnWidth" />
</LinearLayout>
''',
"app/src/main/java/com/hares/monitor/SecurePrefs.kt": r'''
package com.hares.monitor

import android.content.Context
import java.security.MessageDigest
import java.security.SecureRandom

class SecurePrefs(context: Context) {
    private val prefs = context.getSharedPreferences("secure", Context.MODE_PRIVATE)

    fun hasPin() = prefs.contains("pin_hash") && prefs.contains("pin_salt")

    fun setPin(pin: String) {
        val salt = ByteArray(16).also { SecureRandom().nextBytes(it) }
        prefs.edit().putString("pin_salt", salt.hex()).putString("pin_hash", hash(pin, salt)).apply()
    }

    fun verify(pin: String): Boolean {
        val salt = prefs.getString("pin_salt", null)?.hexToBytes() ?: return false
        val expected = prefs.getString("pin_hash", null) ?: return false
        return MessageDigest.isEqual(hash(pin, salt).toByteArray(), expected.toByteArray())
    }

    private fun hash(pin: String, salt: ByteArray): String {
        val md = MessageDigest.getInstance("SHA-256")
        repeat(120_000) { md.update(salt); md.update(pin.toByteArray(Charsets.UTF_8)) }
        return md.digest().hex()
    }

    private fun ByteArray.hex() = joinToString("") { "%02x".format(it) }
    private fun String.hexToBytes() = chunked(2).map { it.toInt(16).toByte() }.toByteArray()
}
''',
"app/src/main/java/com/hares/monitor/SettingsStore.kt": r'''
package com.hares.monitor

import android.content.Context

class SettingsStore(context: Context) {
    private val p = context.getSharedPreferences("settings", Context.MODE_PRIVATE)

    var sensitivity: Int get() = p.getInt("sensitivity", 60) set(v) { p.edit().putInt("sensitivity", v).apply() }
    var minArea: Int get() = p.getInt("minArea", 80) set(v) { p.edit().putInt("minArea", v).apply() }
    var cooldownSec: Int get() = p.getInt("cooldown", 4) set(v) { p.edit().putInt("cooldown", v).apply() }
    var performance: Int get() = p.getInt("performance", 1) set(v) { p.edit().putInt("performance", v).apply() }
    var sound: Boolean get() = p.getBoolean("sound", true) set(v) { p.edit().putBoolean("sound", v).apply() }
    var screenFlash: Boolean get() = p.getBoolean("screenFlash", true) set(v) { p.edit().putBoolean("screenFlash", v).apply() }
    var notification: Boolean get() = p.getBoolean("notification", true) set(v) { p.edit().putBoolean("notification", v).apply() }
    var saveImages: Boolean get() = p.getBoolean("saveImages", true) set(v) { p.edit().putBoolean("saveImages", v).apply() }
    var dimScreen: Boolean get() = p.getBoolean("dimScreen", false) set(v) { p.edit().putBoolean("dimScreen", v).apply() }
    var chargeOnly: Boolean get() = p.getBoolean("chargeOnly", false) set(v) { p.edit().putBoolean("chargeOnly", v).apply() }
    var volume: Int get() = p.getInt("volume", 80) set(v) { p.edit().putInt("volume", v).apply() }
}
''',
"app/src/main/java/com/hares/monitor/OverlayView.kt": r'''
package com.hares.monitor

import android.content.Context
import android.graphics.Canvas
import android.graphics.Paint
import android.graphics.RectF
import android.util.AttributeSet
import android.view.View

class OverlayView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null
) : View(context, attrs) {
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
        style = Paint.Style.STROKE
        strokeWidth = 5f
        color = 0xFF00FF55.toInt()
    }
    @Volatile private var boxes: List<RectF> = emptyList()

    fun setBoxes(newBoxes: List<RectF>) {
        boxes = newBoxes
        postInvalidateOnAnimation()
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        boxes.forEach { canvas.drawRect(it, paint) }
    }
}
''',
"app/src/main/java/com/hares/monitor/MotionDetector.kt": r'''
package com.hares.monitor

import android.graphics.RectF
import kotlin.math.abs
import kotlin.math.max
import kotlin.math.min

data class MotionBox(val rect: RectF, val score: Int)

class MotionDetector(
    private val sensitivity: Int,
    private val minArea: Int
) {
    private var previous: ByteArray? = null
    private var previousW = 0
    private var previousH = 0

    fun detect(gray: ByteArray, width: Int, height: Int): List<MotionBox> {
        val old = previous
        if (old == null || previousW != width || previousH != height) {
            previous = gray.clone()
            previousW = width; previousH = height
            return emptyList()
        }

        // Downsampled grayscale diff. Lower threshold = more sensitive.
        val threshold = max(4, 40 - (sensitivity * 34 / 100))
        val step = 4
        val gw = width / step
        val gh = height / step
        val active = BooleanArray(gw * gh)

        for (y in 0 until gh) {
            val sy = y * step
            for (x in 0 until gw) {
                val sx = x * step
                val idx = sy * width + sx
                val d = abs((gray[idx].toInt() and 255) - (old[idx].toInt() and 255))
                active[y * gw + x] = d >= threshold
            }
        }

        val visited = BooleanArray(active.size)
        val result = ArrayList<MotionBox>()
        val qx = IntArray(active.size)
        val qy = IntArray(active.size)

        for (sy in 0 until gh) for (sx in 0 until gw) {
            val start = sy * gw + sx
            if (!active[start] || visited[start]) continue
            var head = 0; var tail = 0
            qx[tail] = sx; qy[tail] = sy; tail++
            visited[start] = true
            var minX = sx; var maxX = sx; var minY = sy; var maxY = sy; var count = 0

            while (head < tail) {
                val x = qx[head]; val y = qy[head]; head++; count++
                minX = min(minX, x); maxX = max(maxX, x)
                minY = min(minY, y); maxY = max(maxY, y)
                for (dy in -1..1) for (dx in -1..1) {
                    if (dx == 0 && dy == 0) continue
                    val nx = x + dx; val ny = y + dy
                    if (nx !in 0 until gw || ny !in 0 until gh) continue
                    val ni = ny * gw + nx
                    if (active[ni] && !visited[ni]) {
                        visited[ni] = true
                        qx[tail] = nx; qy[tail] = ny; tail++
                    }
                }
            }

            if (count * step * step >= minArea) {
                result += MotionBox(
                    RectF(
                        (minX * step).toFloat(),
                        (minY * step).toFloat(),
                        ((maxX + 1) * step).toFloat(),
                        ((maxY + 1) * step).toFloat()
                    ),
                    count
                )
            }
        }

        previous = gray.clone()
        return mergeNearby(result)
    }

    private fun mergeNearby(input: List<MotionBox>): List<MotionBox> {
        val out = ArrayList<MotionBox>()
        for (b in input.sortedByDescending { it.score }) {
            var merged = false
            for (i in out.indices) {
                val a = out[i].rect
                val expanded = RectF(a.left - 24, a.top - 24, a.right + 24, a.bottom + 24)
                if (expanded.intersects(b.rect.left, b.rect.top, b.rect.right, b.rect.bottom)) {
                    a.union(b.rect)
                    merged = true
                    break
                }
            }
            if (!merged) out += MotionBox(RectF(b.rect), b.score)
        }
        return out.take(12)
    }
}
''',
"app/src/main/java/com/hares/monitor/MonitoringService.kt": r'''
package com.hares.monitor

import android.app.*
import android.content.Context
import android.content.Intent
import android.graphics.RectF
import android.os.Build
import android.os.IBinder
import androidx.camera.core.*
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.core.app.NotificationCompat
import androidx.core.content.ContextCompat
import androidx.lifecycle.LifecycleService
import java.util.concurrent.Executors
import java.util.concurrent.atomic.AtomicBoolean
import kotlin.math.max

class MonitoringService : LifecycleService() {
    companion object {
        const val ACTION_BOXES = "com.hares.monitor.BOXES"
        const val EXTRA_BOXES = "boxes"
        const val ACTION_EVENT = "com.hares.monitor.EVENT"
        const val CHANNEL = "monitoring"
        var running = AtomicBoolean(false)
    }

    private val executor = Executors.newSingleThreadExecutor()
    private lateinit var detector: MotionDetector
    private var lastAlert = 0L
    private var cameraProvider: ProcessCameraProvider? = null

    override fun onCreate() {
        super.onCreate()
        createChannel()
        startForeground(1001, notification("المراقبة نشطة"))
        val s = SettingsStore(this)
        detector = MotionDetector(s.sensitivity, s.minArea)
        running.set(true)
        bindCamera()
    }

    private fun bindCamera() {
        val future = ProcessCameraProvider.getInstance(this)
        future.addListener({
            cameraProvider = future.get()
            val analysis = ImageAnalysis.Builder()
                .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
                .setImageQueueDepth(1)
                .build()
            analysis.setAnalyzer(executor) { image ->
                analyze(image)
            }
            val selector = CameraSelector.DEFAULT_BACK_CAMERA
            try {
                cameraProvider?.unbindAll()
                cameraProvider?.bindToLifecycle(this, selector, analysis)
            } catch (_: Exception) {}
        }, ContextCompat.getMainExecutor(this))
    }

    private fun analyze(image: ImageProxy) {
        val plane = image.planes[0]
        val buffer = plane.buffer
        val bytes = ByteArray(buffer.remaining())
        buffer.get(bytes)
        val w = image.width
        val h = image.height
        val boxes = detector.detect(bytes, w, h).map { it.rect }

        val packed = boxes.flatMap { listOf(it.left, it.top, it.right, it.bottom) }.toFloatArray()
        sendBroadcast(Intent(ACTION_BOXES).setPackage(packageName).putExtra(EXTRA_BOXES, packed))

        if (boxes.isNotEmpty()) {
            val now = System.currentTimeMillis()
            val cooldown = SettingsStore(this).cooldownSec * 1000L
            if (now - lastAlert >= cooldown) {
                lastAlert = now
                sendBroadcast(Intent(ACTION_EVENT).setPackage(packageName))
                if (SettingsStore(this).notification) {
                    val n = notification("تم اكتشاف حركة")
                    (getSystemService(NOTIFICATION_SERVICE) as NotificationManager).notify(1002, n)
                }
            }
        }
        image.close()
    }

    private fun notification(text: String): Notification =
        NotificationCompat.Builder(this, CHANNEL)
            .setSmallIcon(R.drawable.ic_hare)
            .setContentTitle("🛡️ حارس")
            .setContentText(text)
            .setOngoing(text == "المراقبة نشطة")
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .build()

    private fun createChannel() {
        if (Build.VERSION.SDK_INT >= 26) {
            val c = NotificationChannel(CHANNEL, "مراقبة حارس", NotificationManager.IMPORTANCE_HIGH)
            getSystemService(NotificationManager::class.java).createNotificationChannel(c)
        }
    }

    override fun onDestroy() {
        running.set(false)
        cameraProvider?.unbindAll()
        executor.shutdownNow()
        super.onDestroy()
    }

    override fun onBind(intent: Intent): IBinder? = super.onBind(intent)
}
''',
"app/src/main/java/com/hares/monitor/PinActivity.kt": r'''
package com.hares.monitor

import android.content.Intent
import android.os.Bundle
import android.widget.*
import androidx.appcompat.app.AppCompatActivity

class PinActivity : AppCompatActivity() {
    private lateinit var secure: SecurePrefs
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_pin)
        secure = SecurePrefs(this)
        val input = findViewById<EditText>(R.id.pinInput)
        val title = findViewById<TextView>(R.id.pinTitle)
        val button = findViewById<Button>(R.id.pinButton)
        val error = findViewById<TextView>(R.id.pinError)

        if (secure.hasPin()) {
            title.text = "أدخل رمز PIN"
            button.text = "دخول"
        } else {
            title.text = "أنشئ رمز PIN من 6 أرقام"
            button.text = "إنشاء"
        }

        button.setOnClickListener {
            val pin = input.text.toString()
            if (pin.length != 6) {
                error.text = "يجب أن يكون الرمز 6 أرقام"
                return@setOnClickListener
            }
            if (!secure.hasPin()) {
                secure.setPin(pin)
                openMain()
            } else if (secure.verify(pin)) {
                openMain()
            } else {
                error.text = "رمز PIN غير صحيح"
                input.text.clear()
            }
        }
    }

    private fun openMain() {
        startActivity(Intent(this, MainActivity::class.java))
        finish()
    }
}
''',
"app/src/main/java/com/hares/monitor/MainActivity.kt": r'''
package com.hares.monitor

import android.Manifest
import android.content.*
import android.content.pm.PackageManager
import android.os.Bundle
import android.provider.Settings
import android.view.WindowManager
import android.widget.*
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.camera.core.*
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.camera.view.PreviewView
import androidx.core.content.ContextCompat
import android.graphics.RectF

class MainActivity : AppCompatActivity() {
    private lateinit var preview: PreviewView
    private lateinit var overlay: OverlayView
    private lateinit var status: TextView
    private lateinit var startStop: Button
    private var front = false

    private val permissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { startMonitoringIfReady() }

    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            when (intent.action) {
                MonitoringService.ACTION_BOXES -> {
                    val a = intent.getFloatArrayExtra(MonitoringService.EXTRA_BOXES) ?: floatArrayOf()
                    val boxes = a.toList().chunked(4).filter { it.size == 4 }.map {
                        RectF(it[0], it[1], it[2], it[3])
                    }
                    overlay.setBoxes(boxes)
                }
                MonitoringService.ACTION_EVENT -> flashScreen()
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        preview = findViewById(R.id.preview)
        overlay = findViewById(R.id.overlay)
        status = findViewById(R.id.status)
        startStop = findViewById(R.id.startStop)

        startPreview()
        startStop.setOnClickListener {
            if (MonitoringService.running.get()) stopMonitoring() else requestPermissionsAndStart()
        }
        findViewById<Button>(R.id.cameraSwitch).setOnClickListener {
            front = !front
            startPreview()
        }
        findViewById<Button>(R.id.galleryButton).setOnClickListener {
            startActivity(Intent(this, GalleryActivity::class.java))
        }
        findViewById<Button>(R.id.settingsButton).setOnClickListener {
            startActivity(Intent(this, SettingsActivity::class.java))
        }

        val filter = IntentFilter().apply {
            addAction(MonitoringService.ACTION_BOXES)
            addAction(MonitoringService.ACTION_EVENT)
        }
        ContextCompat.registerReceiver(this, receiver, filter, ContextCompat.RECEIVER_NOT_EXPORTED)
    }

    private fun requestPermissionsAndStart() {
        val permissions = mutableListOf(Manifest.permission.CAMERA)
        if (android.os.Build.VERSION.SDK_INT >= 33) permissions += Manifest.permission.POST_NOTIFICATIONS
        permissionLauncher.launch(permissions.toTypedArray())
    }

    private fun startMonitoringIfReady() {
        if (checkSelfPermission(Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) return
        val i = Intent(this, MonitoringService::class.java)
        ContextCompat.startForegroundService(this, i)
        status.text = "🟢 المراقبة نشطة"
        startStop.text = "⏹ إيقاف المراقبة"
    }

    private fun stopMonitoring() {
        stopService(Intent(this, MonitoringService::class.java))
        status.text = "🔴 متوقفة"
        startStop.text = "▶ بدء المراقبة"
        overlay.setBoxes(emptyList())
    }

    private fun startPreview() {
        val future = ProcessCameraProvider.getInstance(this)
        future.addListener({
            val provider = future.get()
            val previewUseCase = Preview.Builder().build().also {
                it.setSurfaceProvider(preview.surfaceProvider)
            }
            val selector = if (front) CameraSelector.DEFAULT_FRONT_CAMERA else CameraSelector.DEFAULT_BACK_CAMERA
            try {
                provider.unbindAll()
                provider.bindToLifecycle(this, selector, previewUseCase)
            } catch (_: Exception) {}
        }, ContextCompat.getMainExecutor(this))
    }

    private fun flashScreen() {
        if (!SettingsStore(this).screenFlash) return
        val original = window.attributes.screenBrightness
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
        val old = window.decorView.background
        window.decorView.setBackgroundColor(0xFFFFFFFF.toInt())
        window.decorView.postDelayed({
            window.decorView.setBackgroundColor(0x00000000)
            window.attributes = window.attributes.apply { screenBrightness = original }
            if (old != null) window.decorView.background = old
        }, 300)
    }

    override fun onDestroy() {
        unregisterReceiver(receiver)
        super.onDestroy()
    }
}
''',
"app/src/main/java/com/hares/monitor/SettingsActivity.kt": r'''
package com.hares.monitor

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.os.PowerManager
import android.provider.Settings
import android.widget.*
import androidx.appcompat.app.AppCompatActivity

class SettingsActivity : AppCompatActivity() {
    private lateinit var s: SettingsStore
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)
        s = SettingsStore(this)

        val sensitivity = findViewById<SeekBar>(R.id.sensitivity)
        val minArea = findViewById<SeekBar>(R.id.minArea)
        val cooldown = findViewById<SeekBar>(R.id.cooldown)
        val performance = findViewById<Spinner>(R.id.performance)

        sensitivity.progress = s.sensitivity
        minArea.progress = s.minArea
        cooldown.progress = s.cooldownSec
        performance.adapter = ArrayAdapter(this, android.R.layout.simple_spinner_dropdown_item,
            arrayOf("🔋 توفير البطارية", "⚖️ متوازن", "🚀 عالي الأداء"))
        performance.setSelection(s.performance)

        fun updateLabels() {
            findViewById<TextView>(R.id.sensitivityLabel).text = "الحساسية: ${sensitivity.progress}"
            findViewById<TextView>(R.id.minAreaLabel).text = "الحد الأدنى للحركة: ${minArea.progress}"
            findViewById<TextView>(R.id.cooldownLabel).text = "فترة التهدئة: ${cooldown.progress} ثوانٍ"
        }
        updateLabels()
        sensitivity.setOnSeekBarChangeListener(simple { s.sensitivity = it; updateLabels() })
        minArea.setOnSeekBarChangeListener(simple { s.minArea = it; updateLabels() })
        cooldown.setOnSeekBarChangeListener(simple { s.cooldownSec = it; updateLabels() })
        performance.onItemSelectedListener = object : android.widget.AdapterView.OnItemSelectedListener {
            override fun onItemSelected(p: android.widget.AdapterView<*>?, v: android.view.View?, pos: Int, id: Long) { s.performance = pos }
            override fun onNothingSelected(p: android.widget.AdapterView<*>?) {}
        }

        bind(R.id.soundEnabled) { s.sound = it }
        bind(R.id.screenFlash) { s.screenFlash = it }
        bind(R.id.notificationEnabled) { s.notification = it }
        bind(R.id.saveImages) { s.saveImages = it }
        bind(R.id.dimScreen) { s.dimScreen = it }
        bind(R.id.chargeOnly) { s.chargeOnly = it }
        findViewById<SeekBar>(R.id.volume).progress = s.volume
        findViewById<SeekBar>(R.id.volume).setOnSeekBarChangeListener(simple { s.volume = it })

        findViewById<Button>(R.id.batteryButton).setOnClickListener {
            val pm = getSystemService(PowerManager::class.java)
            if (!pm.isIgnoringBatteryOptimizations(packageName)) {
                startActivity(Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS,
                    Uri.parse("package:$packageName")))
            } else Toast.makeText(this, "تحسين البطارية متوقف لهذا التطبيق", Toast.LENGTH_SHORT).show()
        }
        findViewById<Button>(R.id.changePin).setOnClickListener {
            startActivity(Intent(this, PinActivity::class.java).putExtra("change", true))
        }
    }

    private fun bind(id: Int, setter: (Boolean) -> Unit) {
        val sw = findViewById<Switch>(id)
        sw.isChecked = when(id) {
            R.id.soundEnabled -> s.sound
            R.id.screenFlash -> s.screenFlash
            R.id.notificationEnabled -> s.notification
            R.id.saveImages -> s.saveImages
            R.id.dimScreen -> s.dimScreen
            else -> s.chargeOnly
        }
        sw.setOnCheckedChangeListener { _, checked -> setter(checked) }
    }

    private fun simple(block: (Int) -> Unit) = object : SeekBar.OnSeekBarChangeListener {
        override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) { block(progress) }
        override fun onStartTrackingTouch(seekBar: SeekBar?) {}
        override fun onStopTrackingTouch(seekBar: SeekBar?) {}
    }
}
''',
"app/src/main/java/com/hares/monitor/GalleryActivity.kt": r'''
package com.hares.monitor

import android.os.Bundle
import android.os.Environment
import android.widget.*
import androidx.appcompat.app.AppCompatActivity
import java.io.File

class GalleryActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_gallery)
        val grid = findViewById<GridView>(R.id.grid)
        val dir = File(getExternalFilesDir(Environment.DIRECTORY_PICTURES), "Hares")
        val images = dir.listFiles()?.filter { it.extension.lowercase() in setOf("jpg","jpeg","png") }?.sortedDescending() ?: emptyList()
        val adapter = object : BaseAdapter() {
            override fun getCount() = images.size
            override fun getItem(position: Int) = images[position]
            override fun getItemId(position: Int) = position.toLong()
            override fun getView(position: Int, convertView: android.view.View?, parent: android.view.ViewGroup): android.view.View {
                val iv = (convertView as? ImageView) ?: ImageView(this@GalleryActivity).apply {
                    layoutParams = AbsListView.LayoutParams(260, 260)
                    scaleType = ImageView.ScaleType.CENTER_CROP
                }
                iv.setImageURI(android.net.Uri.fromFile(images[position]))
                iv.setOnLongClickListener {
                    images[position].delete()
                    recreate()
                    true
                }
                return iv
            }
        }
        grid.adapter = adapter
    }
}
''',
".github/workflows/build-apk.yml": r'''
name: Build Hares APK

on:
  workflow_dispatch:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Build release APK
        run: gradle :app:assembleRelease --no-daemon

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Hares-release-apk
          path: app/build/outputs/apk/release/app-release.apk
''',
"README_AR.md": r'''
# 🛡️ حارس — مراقب الحركة

مشروع Android/Kotlin يستهدف Android 16 (SDK 36).

## المزايا الأساسية
- كشف حركة محلي باستخدام Frame Differencing.
- تحديد مناطق الحركة بمربعات خضراء.
- تتبع متعدد للمناطق المتحركة.
- إعدادات الحساسية والحد الأدنى للحركة وCooldown.
- مستويات أداء: توفير / متوازن / عالي.
- Foreground Service للكاميرا.
- إشعار مراقبة دائم.
- واجهة عربية.
- PIN من 6 أرقام.
- تبديل الكاميرا الأمامية والخلفية.
- وميض الشاشة بدل فلاش الكاميرا.
- إعدادات البطارية.
- معرض محلي.

## ملاحظة عن Android 16
Android يفرض قيودًا على الكاميرا وخدمات الخلفية. لا يمكن ضمان أن الكاميرا ستستمر مع الشاشة مطفأة على كل جهاز. يجب تشغيل خدمة الكاميرا من واجهة ظاهرة وبالصلاحيات المطلوبة.

## بناء APK
يمكن البناء محليًا باستخدام Gradle 8.9 وJDK 17:
`gradle :app:assembleRelease`

أو ارفع المشروع إلى GitHub وشغّل:
Actions → Build Hares APK → Run workflow

سيظهر `app-release.apk` في Artifacts.
'''
}

for rel, content in files.items():
    p = root / rel
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(textwrap.dedent(content).lstrip(), encoding="utf-8")

zip_path = Path("/mnt/data/Hares-Android16-Project.zip")
if zip_path.exists():
    zip_path.unlink()
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for p in root.rglob("*"):
        if p.is_file():
            z.write(p, p.relative_to(root.parent))

print(f"تم إنشاء مشروع حارس: {zip_path}")
print(f"عدد الملفات: {sum(1 for p in root.rglob('*') if p.is_file())}")

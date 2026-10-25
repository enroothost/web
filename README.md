# ParentGuard - Web Panel & Android Connection Setup

Ye project ek complete Parental Control System ka **Web Panel** hai jo **Render** par host hone ke liye ready hai, aur **Firebase Firestore** ke sath real-time me connect hota hai.

---

## 1. Render.com par Kaise Host Karein?

1. Apne computer par is directory me git repo banayein:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```
2. Is repository ko **GitHub** par push karein.
3. **[Render.com](https://render.com)** par login karein aur **"New +" -> "Web Service"** select karein.
4. Apni GitHub repository choose karein.
5. Ye settings configure karein:
   * **Name:** `parentguard-web`
   * **Runtime:** `Node`
   * **Build Command:** `npm install`
   * **Start Command:** `npm start`
   * **Instance Type:** `Free`
6. Click **Deploy Web Service**. Aapko live URL mil jayega (jaise: `https://parentguard-web.onrender.com`).

---

## 2. Firebase Setup (1-Time)

1. **[Firebase Console](https://console.firebase.google.com/)** me naya project banayein: `parent-guard-app`.
2. **Firestore Database** enable karein:
   * "Create Database" -> "Start in test mode" (testing ke liye).
3. **Web App Add Karein:**
   * Project settings me jaakar Web icon `</>` par click karein.
   * `firebaseConfig` object copy karein.
4. Render par live website open karein -> **"Firebase Settings"** par click karein aur apna JSON paste kar dein.

---

## 3. Child Android App (APK Side) Setup

Android Studio me aapka Android project isi same Firebase Firestore se connect hoga:

### A. Dependencies (`build.gradle.kts` / `build.gradle`):
```groovy
implementation platform('com.google.firebase:firebase-bom:33.0.0')
implementation 'com.google.firebase:firebase-firestore'
```

### B. Data Sync Karne Ka Kotlin Code (Child Device se Firestore):
Jab child device open ho ya regular interval par (WorkManager ke through):

```kotlin
import com.google.firebase.firestore.FirebaseFirestore

val db = FirebaseFirestore.getInstance()
val deviceId = "child_phone_01" // Unique Device ID

val deviceData = hashMapOf(
    "deviceName" to "Child's Galaxy Phone",
    "battery" to 82,
    "isOnline" to true,
    "lastActive" to java.time.Instant.now().toString(),
    "dailyUsage" to hashMapOf(
        "YouTube" to "50m",
        "Chrome" to "15m",
        "Minecraft" to "30m"
    )
)

// Firestore me update karna
db.collection("devices").document(deviceId)
    .set(deviceData)
```

### C. Web Panel Se Commands Listen Karna (Remote Lock/Action):
Android app me Firestore ka listener lagaya jata hai jo Web Panel se aane wale changes ko turant read karta hai:

```kotlin
db.collection("devices").document(deviceId)
    .addSnapshotListener { snapshot, e ->
        if (snapshot != null && snapshot.exists()) {
            val isLocked = snapshot.getBoolean("isLocked") ?: false
            if (isLocked) {
                // Parent ne Web Panel se Lock daba diya:
                // Yahan lock overlay ya restriction screen dikhayi jaati hai
            }
        }
    }
```

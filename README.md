# NFC Patrols — Secure-Site Patrol Monitoring POC

This proof of concept responds to the **“NFC Patch Patrol Monitoring”** call for tenders.

A security guard scans an NFC patch using an Android phone, while the security control computer displays the status of all checkpoints and updates them in real time through WebSocket communication.

Two scanning clients are available for the demonstration:

* **Native Android application** (`android/`) — recommended for the final demonstration. It does not require HTTPS and can be installed directly on a phone using USB and `adb`.
* **Mobile web page** (`scan.html`, using Web NFC) — requires no installation but requires HTTPS. It is also useful for the enrollment page (`enroll.html`) and the PC dashboard.

## Technology Stack

* **Backend:** Kotlin with [Ktor](https://ktor.io), using a Netty server.
* **Database:** [Exposed](https://github.com/JetBrains/Exposed) with a local H2 database.
* **Mobile NFC scanning:** Native Android application using `NfcAdapter.enableReaderMode`, or Web NFC using `NDEFReader`.
* **NFC patch identifier:** Both clients send the same `tagUid`, corresponding to the physical tag UID formatted as hexadecimal values separated by colons.
* **Real-time monitoring:** WebSocket communication after each scan, combined with a browser-side timer that updates the elapsed time.
* **Web frontend:** Vanilla HTML, CSS, and JavaScript served directly by Ktor from `src/main/resources/static`.

See `docs/ARCHITECTURE.md` for a detailed explanation of the technical choices.

## Starting the Server

### Requirements

JDK 21 is required.

A JDK installation has already been detected at:

```text
C:\Program Files\Java\jdk-21
```

### Build and Run

```bash
mvn -DskipTests package
java -jar target/rondes-nfc.jar
```

The server runs at:

```text
http://localhost:8080
```

The port can be changed using the `PORT` environment variable.

The H2 database is created during the first startup at:

```text
./data/rondes-nfc.mv.db
```

It is automatically populated with demonstration data when the database is empty.

To reset the application and recreate the demonstration data, delete the `data/` directory before restarting the server.

## Demonstration Accounts

The following accounts are created by `Seed.kt` when the guards table is empty:

| Badge  | Full name      |    PIN | Role            |
| ------ | -------------- | -----: | --------------- |
| `beso` | Beso Gogoladze | `1234` | `DIRECTION`     |
| `CP01` | Chef de poste  | `1111` | `CHEF_DE_POSTE` |
| `G002` | Gardien 2      | `2222` | `GARDIEN`       |
| `G003` | Gardien Dubois | `3333` | `GARDIEN`       |

The PIN codes are not stored directly in the database. They are hashed using `AuthService.hashPin()` before being saved.

## Preconfigured Checkpoints

Five checkpoints are created by `Seed.kt` when the rooms table is empty:

| Checkpoint | Building               | Floor | Orange alert |   Red alert |
| ---------- | ---------------------- | ----- | -----------: | ----------: |
| `Point 1`  | `Batiment d'avocat`    | `RDC` |   60 minutes | 120 minutes |
| `Point 2`  | `Batiment La Poste`    | `RDC` |   90 minutes | 180 minutes |
| `Point 3`  | `Maison M.Chamoulaud`  | `RDC` |  180 minutes | 360 minutes |
| `Point 4`  | `Batiment Print`       | `RDC` |   60 minutes | 150 minutes |
| `Point 5`  | `Batiment Infirmierie` | `RDC` |  120 minutes | 240 minutes |

The building and floor names shown above correspond exactly to the values stored by `Seed.kt`.

No NFC patch is assigned to a checkpoint in advance. The patch-to-checkpoint association must be performed during the demonstration through the `enroll.html` page.

This live enrollment process is part of the checkpoint and NFC patch management functionality.

## Application Pages

| URL               | Purpose                                                          | Authorized roles                                |
| ----------------- | ---------------------------------------------------------------- | ----------------------------------------------- |
| `/index.html`     | Main navigation portal                                           | All roles                                       |
| `/login.html`     | Badge and PIN authentication                                     | All roles                                       |
| `/scan.html`      | Scan a checkpoint NFC patch from a mobile device                 | `GARDIEN`, `CHEF_DE_POSTE`, `DIRECTION`         |
| `/enroll.html`    | Assign an NFC patch to a checkpoint                              | `CHEF_DE_POSTE`, `DIRECTION`                    |
| `/dashboard.html` | Real-time monitoring dashboard for the security control computer | `CHEF_DE_POSTE`, `DIRECTION`                    |
| `/history.html`   | View patrol scan history                                         | All roles; guards can only view their own scans |

## Android Application

The recommended scanning client for the demonstration is the native Android application.

Its source code is located in the `android/` directory.

It is an independent Gradle project and is separate from the Maven backend project. It should therefore be opened separately in Android Studio or built using the command line.

### Why a Native Android Application Is Included

The native Android application uses:

```text
NfcAdapter.enableReaderMode
```

Unlike Web NFC, the Android API does not require a secure HTTPS context.

The application can communicate directly with the backend through the local IP address of the security control computer:

```text
http://<PC-IP>:8080
```

This avoids the need to configure an HTTPS tunnel or certificate during the demonstration.

For this reason, the native Android application is the most reliable solution for the final presentation.

### Build and Install the Android Application

```bash
cd android
./gradlew assembleDebug
```

The generated APK is available at:

```text
app/build/outputs/apk/debug/app-debug.apk
```

Install it on the connected Android phone with:

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Alternatively:

1. Open the `android/` directory in Android Studio.
2. Connect the Android phone using USB.
3. Enable USB debugging on the phone.
4. Select the phone as the deployment device.
5. Start the application using the **Run** button.

## Using the Android Application

1. Start the backend on the security control computer:

   ```bash
   java -jar target/rondes-nfc.jar
   ```

2. Find the computer’s local IPv4 address:

   ```bash
   ipconfig
   ```

   Example:

   ```text
   192.168.1.42
   ```

3. Open the Android application.

4. Enter the backend address:

   ```text
   http://192.168.1.42:8080
   ```

5. Sign in using one of the demonstration accounts.

6. Move the phone close to an NFC patch that has already been assigned to a checkpoint through `enroll.html`.

7. The application immediately displays the scanned checkpoint name.

8. The monitoring dashboard at `/dashboard.html` is updated in real time.

The Android application and the Web NFC client use the same tag identifier format: the physical NFC UID represented in hexadecimal and separated by colons.

Therefore, an NFC patch enrolled through one client is recognized by the other client without additional configuration.

### Android Application Limitation

The Android application is a demonstration application.

It does not include:

* A signed production release build.
* Distribution through an application store.
* A permanent production security configuration.

The following option is enabled in the Android manifest:

```xml
android:usesCleartextTraffic="true"
```

This allows the application to communicate with the backend over HTTP on the local network.

In production, clear-text communication should be restricted to the internal server address or replaced with HTTPS and TLS.

See `docs/SECURITY.md` for more information.

## Live Demonstration with Web NFC

The `scan.html` page provides an alternative scanning solution that does not require application installation.

It uses:

```javascript
navigator.NDEFReader
```

However, Web NFC requires a secure context:

* `https://`
* `http://localhost`

An Android phone opening the backend through the computer’s local IP address, for example:

```text
http://192.168.1.42:8080
```

is not considered to be using a secure context.

As a result, NFC scanning will not be activated in the browser.

The native Android application does not have this limitation.

### Option 1: HTTPS Tunnel

The simplest solution is to use ngrok or Cloudflare Tunnel.

Example with ngrok:

```bash
ngrok http 8080
```

Open the generated HTTPS address from the Android phone:

```text
https://xxxx.ngrok-free.app
```

Requirements:

* Use Chrome on Android.
* Enable NFC in the phone settings.
* Allow the website to use NFC when requested.

This option requires an internet connection but does not require server-side certificate configuration.

### Option 2: Local Network with a Trusted Certificate

When no internet connection is available during the demonstration, a local trusted certificate can be generated using `mkcert`.

The certificate must then be:

1. Loaded into Ktor using an `sslConnector`.
2. Installed as a trusted root certificate on the demonstration phone.
3. Used to access the server through HTTPS on the local network.

This solution requires more preparation but does not depend on an external tunneling service.

## Demonstration Fallback

When Web NFC is unavailable, the `scan.html` and `enroll.html` pages display a manual UID input option.

This fallback can be used to demonstrate the complete workflow without a physical NFC scan in case of:

* Network failure.
* Browser compatibility problems.
* NFC hardware problems.
* Damaged or unreadable NFC patches.

The manual UID input must not remain enabled in a production environment.

See `docs/SECURITY.md` for the related security recommendations.

## Implemented Requirements

### Must-Have Features

* Real NFC patch scanning through the Android application or Web NFC.
* Identification of the scanned checkpoint.
* Guard authentication using a badge and PIN.
* Scan timestamp recording.
* Persistent scan storage in the H2 database.
* Real-time monitoring dashboard.
* Display of the elapsed time since the last inspection.
* Display of the last guard who inspected the checkpoint.
* Green, orange, and red checkpoint statuses.
* Automatic dashboard updates using WebSocket.

### Should-Have Features

* Checkpoint CRUD operations.
* NFC patch-to-checkpoint assignment through `enroll.html`.
* Room API through `/api/rooms`.
* NFC patch API through `/api/patches`.
* Filterable scan history.
* Filtering by checkpoint.
* Filtering by guard.
* Filtering by date or period.
* History page through `history.html`.
* History API through `/api/history`.
* Role-based accounts:

    * `GARDIEN`
    * `CHEF_DE_POSTE`
    * `DIRECTION`
* Configurable orange and red alert thresholds for each checkpoint.

### Additional Features

* Offline scan queue on the Android phone.
* Delayed synchronization when the network connection is restored.
* Damaged NFC patch reporting.
* A damaged patch report immediately forces the checkpoint status to red.
* Account locking after five failed PIN attempts.

## Known POC Limitations

The planned improvements are documented in `docs/OFFRE.md`.

* Mobile NFC scanning is supported on Android only.
* Native Android NFC scanning and Web NFC are not supported on iOS.
* Possible iOS alternatives are described in the roadmap.
* The Android application only handles checkpoint scanning.
* NFC patch enrollment remains available through the web interface.
* The monitoring dashboard remains available through the web interface.
* The security control computer does not require a mobile application.
* The scan is not verified using geolocation.
* A guard must physically scan an NFC patch, but the system cannot detect whether the patch has been removed from its original location.
* Web NFC uses an HTTPS tunnel during the demonstration.
* The native Android application uses clear-text HTTP communication on the local network.
* A permanent internal TLS certificate is not included in the one-week POC.
* Production deployment would require stronger authentication, network security, device management, backups, monitoring, and a permanent TLS configuration.

# Risk Analysis — NFC Security Patrols

A patrol-control system that can be easily deceived has no operational value. Its purpose is to provide evidence that an authorised security guard was physically present at a specific checkpoint.

Each threat below is therefore treated as a design requirement, not as a simple compliance checkbox.

## 1. Patrol Validation Fraud

| Threat                                                                                    | Analysis                                                                                                                                                                                                                                                                                                                                                | Response                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A security guard validates a checkpoint from home or without being physically present** | Under normal application use, this is not possible because the scan is a physical Web NFC event triggered through `NDEFReader`. The smartphone must be placed within a few centimetres of the NFC tag. A browser cannot remotely generate a real NFC reading event.                                                                                     | The real weakness is not the smartphone—it is the **API**. A technically skilled person who knows a valid `tagUid` could attempt to call `POST /api/scan` directly using a tool such as `curl`, without performing a genuine NFC scan. Authentication using badge ID and PIN, short-lived session tokens and detailed logging of IP address, user agent and device information are therefore required. Anomaly detection is also essential. A valid session token alone does not prove physical presence. |
| **An NFC checkpoint is cloned by copying its UID to another tag**                         | The NTAG213, NTAG215 and NTAG216 tags recommended for the POC use a factory-programmed UID that is not normally rewritable. This differs from some specialised “magic” tags designed to imitate certain older NFC technologies. Cloning an NTAG21x UID generally requires specialised hardware and is beyond the capabilities of an opportunistic user. | For the POC, the physical UID is considered sufficient. For a production roadmap requiring a stronger level of evidence, the project could migrate to secure tags such as NTAG424 DNA using SUN or SDM. These tags generate cryptographically protected data that changes with each read, reducing the risk of replaying or copying a previously captured identifier.                                                                                                                                     |
| **An intercepted request is replayed repeatedly using the same `tagUid` and token**       | Replaying the same request would create several scans with very close timestamps. These repeated validations would remain visible in the patrol history and would be operationally suspicious.                                                                                                                                                          | The dashboard and history must never hide duplicate validations. For example, if the same security guard scans the same checkpoint ten times in two minutes, the security supervisor should be able to identify the anomaly. Automatic duplicate-detection thresholds can be added as a production enhancement.                                                                                                                                                                                           |
| **A guard shares their badge ID and PIN with another person**                             | The system would record the correct account but not necessarily the correct physical person. This is a general limitation of shared credentials.                                                                                                                                                                                                        | Enforce individual accounts, prohibit credential sharing, log device and session information, apply disciplinary procedures and consider stronger authentication such as device registration, biometrics or a second factor for higher-security deployments.                                                                                                                                                                                                                                              |
| **A guard manually changes the smartphone date and time before an offline scan**          | Offline scans rely partly on the timestamp produced by the client device. A manipulated device clock could produce an incorrect `scanned_at` value.                                                                                                                                                                                                     | The server stores both `scanned_at` and `received_at`, accepts offline timestamps only within a limited time window and flags suspicious differences for review. Production versions should also record device-clock drift and reject impossible or inconsistent sequences.                                                                                                                                                                                                                               |

## 2. Field Availability

| Threat                                                                                             | Analysis                                                                                                                                                                                             | Response                                                                                                                                                                                                                                                                                                                                                                                             |
| -------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **There is no network connection in a basement or restricted area when the checkpoint is scanned** | This case is explicitly included in the client requirement. NFC reading itself is local and does not require an internet connection. Only transmission to the backend requires network availability. | `scan.html` records the timestamp at the moment of the physical NFC tap using `new Date().toISOString()`. It then attempts to send the scan to the server. When the request fails because of a network error, the scan is placed in a local queue using `localStorage`. The queue is automatically retried when the browser receives an `online` event and during periodic synchronisation attempts. |
| **An offline scan is transmitted much later**                                                      | A delayed scan may be legitimate, but accepting any client-provided timestamp without limits would allow arbitrary backdating.                                                                       | The server accepts a past scan only within a controlled window, set to six hours by default through `ScanService.MAX_OFFLINE_WINDOW`. Scans outside this period are rejected and must be reviewed manually. The dashboard and history clearly identify delayed scans as **offline synchronisations**. Transparency is preferred over silently accepting all delayed data.                            |
| **The guard’s smartphone fails or runs out of battery during the patrol**                          | This is partly outside the software scope. However, the absence of expected scans remains visible in the supervision system.                                                                         | When a checkpoint is not validated within its expected period, its status changes from green to orange and then red. The security supervisor can react during the shift instead of discovering the missing patrol the following day. A charged replacement device should also be available at the security control station.                                                                          |
| **The application server becomes unavailable**                                                     | Guards may still be able to read NFC tags, but scans cannot be transmitted and the control station loses real-time visibility.                                                                       | Store scans locally, monitor server availability, use automated restarts, configure health checks and maintain backups. For high-availability environments, deploy several backend instances behind a load balancer.                                                                                                                                                                                 |
| **The browser is closed before queued scans are synchronised**                                     | Pending scans stored in `localStorage` remain on the device, but they are not transmitted until the application is opened again.                                                                     | Display the number of pending scans clearly, automatically retry on page load and instruct guards not to end their shift while scans remain unsynchronised. A production native application or service worker could provide stronger background synchronisation.                                                                                                                                     |

## 3. Physical Integrity of the Secured Site

| Threat                                                                          | Analysis                                                                                                                                                                                                                                                 | Response                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **An NFC patch is removed, destroyed or stolen**                                | Two cases must be distinguished. First, no one reports the damage: the absence of future scans naturally causes the checkpoint status to move from green to orange and then red. Second, a security guard notices the damage and reports it immediately. | The endpoint `POST /api/patches/{id}/report-damaged` can be used by any authenticated user. It forces the associated checkpoint into a red state even when a recent valid scan exists. This prevents a previous successful control from hiding a current hardware problem. Only a `CHEF_DE_POSTE` or `DIRECTION` user may clear the damaged status, normally after physical replacement and reassignment through `enroll.html`. |
| **A patch is removed from its checkpoint and attached elsewhere**               | A malicious person could carry the NFC patch to another location and validate the checkpoint without visiting the original secured area. This residual threat is not fully addressed by the POC because it requires physical tamper protection.          | The production roadmap should use destructible or tamper-evident labels that cannot be removed intact. Periodic physical inspections should also verify that every tag remains in its authorised position.                                                                                                                                                                                                                      |
| **A photograph or printed UID is used instead of the real checkpoint**          | A printed identifier does not trigger a genuine NFC reading, but it could be used with the demonstration-only manual-entry feature.                                                                                                                      | Manual UID entry must be disabled in production. Only NFC events generated by the reader should be accepted from the production interface.                                                                                                                                                                                                                                                                                      |
| **A checkpoint is hidden, obstructed or made deliberately difficult to access** | Guards may skip the location or report failed readings.                                                                                                                                                                                                  | Checkpoint placement must be validated during site installation. Tags should be accessible, clearly identified, protected from accidental damage and positioned so that scanning requires the guard to reach the intended area.                                                                                                                                                                                                 |
| **Several checkpoints are installed too close together**                        | The phone could read the wrong tag or guards could confuse checkpoint locations.                                                                                                                                                                         | Maintain adequate physical spacing, label checkpoints clearly and display the detected checkpoint name before final validation where operationally appropriate.                                                                                                                                                                                                                                                                 |

## 4. Accounts and Access Control

The system uses role-based permissions to define who can view or modify each category of information.

| Role                                    | Permitted Actions                                                                                                                             | Restricted Actions                                                                                                                   |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **GARDIEN — Security Guard**            | Scan an NFC checkpoint, report a damaged patch and consult their own patrol history.                                                          | Cannot access the global supervision dashboard, view other guards’ activity or manage secured areas, patches and user accounts.      |
| **CHEF_DE_POSTE — Security Supervisor** | Access the real-time dashboard, manage secured areas and NFC patches, consult the complete patrol history and create security-guard accounts. | Cannot create other `CHEF_DE_POSTE` or `DIRECTION` accounts.                                                                         |
| **DIRECTION — Management**              | Has all security-supervisor permissions and can manage accounts, including privileged users.                                                  | Scanning is not part of the normal management role, although it may remain technically available for exceptional control operations. |

This separation directly addresses the client’s question: **who is authorised to see and do what?**

A security guard should not be able to observe:

* the detailed patrol habits of colleagues;
* all checkpoint statuses;
* areas that are checked less frequently;
* security weaknesses across the site;
* administrative configuration;
* other users’ patrol history.

Providing such information to every guard would create an unnecessary map of the site’s operational vulnerabilities.

### Access-Control Requirements

The backend must enforce permissions independently of the user interface.

Hiding a button in HTML is not sufficient. Every protected API endpoint must verify:

* that the session token is valid;
* that the account remains active;
* that the session has not expired;
* that the user has the required role;
* that the requested data belongs to the authorised scope.

For example, a guard requesting `/api/history` must receive only their own entries, even when they manually modify the browser request.

## 5. Authentication and Session Security

### PIN Protection

A four-digit PIN has only 10,000 possible combinations and therefore provides limited entropy.

To reduce brute-force risk:

* the account is locked after five failed login attempts;
* the lock lasts 15 minutes;
* failed-attempt counters are stored by `AuthService`;
* successful authentication resets the failure counter;
* authentication failures are logged for supervision.

This delay makes automated guessing significantly more difficult while avoiding a permanent lockout when a legitimate user enters the wrong PIN a few times.

For higher-security production deployments, a longer PIN or password should be considered.

### Secure PIN Storage

PIN codes are never stored in plaintext.

They are hashed using **bcrypt** through `jbcrypt`. Bcrypt automatically incorporates a salt and is intentionally computationally expensive, making large-scale password guessing slower if the database is compromised.

The application compares the submitted PIN against the stored bcrypt hash during authentication.

### Session Tokens

Sessions use opaque random tokens:

* generated using `SecureRandom`;
* approximately 256 bits in length;
* stored on the server;
* revocable immediately;
* linked to a specific guard account;
* assigned a limited lifetime;
* expired after approximately 12 hours, corresponding to a security shift.

A server-side session can be invalidated immediately when:

* the user logs out;
* the account is disabled;
* suspicious activity is detected;
* the device is lost;
* management revokes access.

The tokens must not be stored in long-lived persistent cookies.

### Transport Security

All production traffic must use:

```text
HTTPS for REST API requests
WSS for WebSocket connections
```

This protects credentials, tokens and patrol data from interception while in transit.

HTTPS is also required by the Web NFC security model in supported browsers.

### Additional Production Controls

A production deployment should also consider:

* session inactivity timeouts;
* explicit logout at the end of each shift;
* device registration;
* automatic revocation after PIN changes;
* rate limiting by account and IP address;
* alerts for repeated failed authentication;
* restricted access from authorised networks or devices;
* mobile-device management for company smartphones.

## 6. Features That Must Not Be Released Unchanged to Production

The POC includes a **manual `tagUid` entry fallback** in `scan.html` and `enroll.html`.

This feature exists only to support the live demonstration when NFC hardware or browser compatibility fails unexpectedly.

Manual entry removes the physical guarantee that the system is designed to provide.

Typing an identifier on a keyboard does not prove that a security guard physically visited the checkpoint.

The feature must therefore:

* be disabled in production;
* be controlled through a build flag or environment variable such as `DEMO_MODE`;
* never be accessible from the production user interface;
* never be exposed outside the controlled demonstration environment;
* be rejected by the production API when physical-proof enforcement is enabled.

Example configuration:

```text
DEMO_MODE=true   → manual UID entry allowed for the presentation
DEMO_MODE=false  → only genuine NFC scan events accepted
```

The production build should use:

```text
DEMO_MODE=false
```

Other POC limitations that must be reviewed before production include:

* use of H2 instead of PostgreSQL;
* incomplete administrator audit logging;
* browser-side storage of offline scans;
* limited automated anomaly detection;
* lack of device registration;
* lack of centralised monitoring;
* absence of formal backup and recovery procedures;
* reliance on standard NFC UID validation rather than cryptographic tags.

## 7. Accepted Roadmap Items Not Yet Implemented

### Temporal Anomaly Detection

The system could detect physically impossible patrol sequences.

For example:

* the same guard validates two distant checkpoints within an unrealistically short period;
* several checkpoints are validated faster than the minimum expected walking time;
* one checkpoint is scanned repeatedly without progression through the patrol route;
* scans occur outside the guard’s assigned shift.

This type of control is sometimes described as **impossible travel detection**.

### Scan Geolocation

The smartphone’s GPS position could be compared with the expected coordinates of the checkpoint.

This may help identify a tag that has been removed and carried elsewhere.

However, GPS has limitations:

* poor indoor accuracy;
* weak reception in basements;
* privacy implications;
* device-permission requirements;
* possible location spoofing.

Geolocation should therefore be treated as an additional signal rather than absolute proof.

### Cryptographic Patrol-History Chaining

Each scan could include the hash of the previous scan.

Example:

```text
scan_hash =
SHA-256(previous_scan_hash + checkpoint_id + guard_id + scanned_at + received_at)
```

Any later modification or deletion in the sequence would break the chain and become detectable.

This approach may be relevant when patrol history requires strong legal or contractual evidential value.

### Administrative Audit Log

The application should maintain an immutable audit trail of sensitive actions, including:

* who created or modified a secured area;
* who enrolled or reassigned an NFC patch;
* who reported or cleared a damaged patch;
* who created, disabled or modified an account;
* who changed checkpoint thresholds;
* who exported patrol history;
* when the action occurred;
* from which device or IP address it was performed.

### Cryptographically Secure NFC Tags

For stronger production security, static UID identification could be replaced by NFC tags capable of producing dynamic cryptographic values.

This would make copied identifiers and replayed reads significantly more difficult.

### Device Registration

Each authorised smartphone could be registered with the backend.

A scan request would then require:

* a valid guard session;
* an approved device identifier;
* a registered NFC checkpoint;
* a valid timestamp;
* an acceptable sequence of patrol activity.

### Patrol Route Validation

The system could define expected patrol routes and verify:

* required checkpoints;
* checkpoint order;
* minimum and maximum route duration;
* missed areas;
* repeated areas;
* shift-specific patrol requirements.

### Security Alerts

Automatic alerts could be generated when:

* a checkpoint becomes overdue;
* several scans fail;
* a patch is reported damaged;
* a guard account is repeatedly locked;
* an impossible patrol sequence is detected;
* a large offline queue is synchronised unexpectedly;
* administrative settings are changed;
* a privileged account logs in from an unknown device.

## 8. Residual Risks

No technical system can completely guarantee that a patrol has been performed correctly.

The system provides strong traceability, but residual risks remain:

* guards may share credentials;
* a tag may be physically relocated;
* a device may be compromised;
* authorised administrators may misuse their permissions;
* offline timestamps may be manipulated;
* physical access does not prove that the area was inspected properly;
* a scan proves proximity to the tag, not the quality of the security check.

The solution must therefore be combined with:

* operational procedures;
* staff training;
* physical inspections;
* management supervision;
* periodic audits;
* clear incident-handling rules;
* appropriate disciplinary and contractual measures.

The NFC system is a tool for strengthening patrol traceability. It does not replace professional security procedures or human supervision.

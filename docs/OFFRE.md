# Commercial Proposal, Delivery Method and Project Schedule — NFC Security Patrols

> The daily rates, unit costs and budget figures below are indicative and should be adjusted according to the actual commercial conditions of the company submitting the proposal. However, the structure and general order of magnitude are representative of a project of this size.

## 1. Selected Delivery Method: Scrum

### Why Scrum Instead of the V-Model

Several parts of the client’s requirements still require clarification, including:

* alert thresholds;
* offline operation;
* user roles and permissions;
* patrol frequency;
* checkpoint criticality;
* escalation procedures.

A traditional V-model requires the specifications to be fully defined and approved before development begins. In this context, that would create a significant risk of misunderstanding requirements that can only be clarified through rapid demonstrations and regular discussions with the client.

Scrum makes it possible to deliver a demonstrable product increment at the end of the first sprint.

The main technical risk—real NFC scanning on the security guards’ target devices—is validated first. This follows an important project principle: NFC compatibility must be tested at the beginning of the project, not at the end.

The remaining SHOULD HAVE features can then be adjusted according to feedback from the pilot site’s security supervisor.

### Team Organisation

| Role                                      | Responsibilities                                                                                                                                         |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Product Owner — Client Representative** | Prioritises the backlog, validates each product increment and makes decisions regarding unclear requirements such as thresholds, roles and patrol rules. |
| **Scrum Master**                          | Facilitates Scrum ceremonies, removes blockers and protects the development team from uncontrolled scope changes during a sprint.                        |
| **Two Backend / Full-Stack Developers**   | Develop the Ktor API, database, Web NFC integration, security dashboard and administration features.                                                     |
| **Test and Acceptance Engineer**          | Writes acceptance scenarios and tests each product increment using real NFC tags and the Android devices selected for deployment.                        |

Development work is intentionally shared between two developers to avoid creating a single point of failure where only one person understands the entire system.

Each feature must be reviewed through a pull request before it is merged into the main branch.

### Scrum Ceremonies and Tools

The proposed organisation includes:

* one- or two-week sprints;
* a 15-minute daily meeting;
* sprint planning at the beginning of each sprint;
* a sprint review and demonstration at the end of each sprint;
* a retrospective to identify improvements for the next sprint.

Project tracking can use Jira or GitHub Projects, depending on the tools already used by the client.

Source code management will use Git with GitHub or GitLab.

Every pull request must be reviewed before merging. A continuous integration pipeline, such as GitHub Actions, will automatically execute unit tests and verify that the application builds successfully after each push.

A dedicated Slack or Microsoft Teams channel should also be created with the client’s Product Owner. This allows the team to quickly clarify operational questions without waiting for the next formal Scrum meeting.

### Testing and User Acceptance Strategy

| Test Level                                    | What Is Tested                                                                                                                                                                       | Tools and Method                                                                                          |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| **Unit testing**                              | Business rules such as alert-level calculation, accepted offline delay, account locking and checkpoint validation rules.                                                             | JUnit 5 and `kotlin-test`                                                                                 |
| **Integration testing**                       | API endpoints for authentication, NFC scans, history, administration and role-based permissions using a test database.                                                               | Ktor test engine with an in-memory H2 database                                                            |
| **Hardware testing**                          | Real NFC reading using several tag models, such as NTAG213, NTAG215 and NTAG216, on the Android devices selected by the client. Android and Chrome fragmentation must be considered. | On-site testing with a compatibility checklist                                                            |
| **User acceptance testing**                   | A real security guard completes an entire patrol round on the pilot secured site using the application.                                                                              | Acceptance checklist based on the MUST HAVE and SHOULD HAVE requirements, formally approved by the client |
| **Load testing before multi-site deployment** | Several dozen scans per minute and multiple simultaneous WebSocket connections.                                                                                                      | k6 or Gatling                                                                                             |

### Project Risk Management

This project risk analysis is separate from the cybersecurity and physical-security risk analysis described in `SECURITY.md`.

| Risk                                                                      | Potential Impact                                            | Mitigation                                                                                                                                                  |
| ------------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Web NFC is incompatible with certain phones or older Android versions** | The main MUST HAVE feature cannot be used.                  | Validate NFC compatibility during Sprint 1 before developing lower-priority features. Provide the client with a list of certified Android devices.          |
| **Some security guards are not comfortable using smartphones**            | Low adoption and incorrectly completed patrols.             | Provide a deliberately simple interface with one main scan button, short on-site training and a temporary paper-based fallback procedure during deployment. |
| **The network is unstable in certain secured areas**                      | Scans may not be transmitted immediately.                   | Use the offline queue already implemented in the POC. Scans are stored locally and automatically synchronised when connectivity returns.                    |
| **The scope changes during a sprint**                                     | Delays and incomplete deliveries.                           | The Product Owner defines sprint content at the start of the sprint. New requests are added to the backlog and evaluated for the next sprint.               |
| **An NFC patch is removed, damaged or copied**                            | Incorrect checkpoint validation or missing patrol evidence. | Use protected tag holders, record damaged tags, conduct regular checks and replace or deactivate compromised tags immediately.                              |
| **A shared security device is lost or stolen**                            | Unauthorised access to patrol functions.                    | Use short-lived sessions, immediate session revocation, PIN-protected guard accounts and remote device-management procedures where available.               |

## 2. Indicative Cost Estimate

### Hardware Costs per Secured Site

The following estimate excludes the existing security control station computer.

| Item                                                               | Indicative Unit Cost |                   Typical Quantity for a Medium Site with 20 Checkpoints |       Estimated Total |
| ------------------------------------------------------------------ | -------------------: | -----------------------------------------------------------------------: | --------------------: |
| NTAG213 adhesive NFC patch                                         |                €0.40 |                                20 checkpoints plus 20% replacement stock |     Approximately €10 |
| Protective or anti-tamper NFC tag holder                           |                   €3 |                                                                       24 |     Approximately €72 |
| NFC-compatible Android smartphone, when not supplied by the client |                 €180 | One per security guard working simultaneously, for example three devices |    Approximately €540 |
| TLS certificate for an internal site                               |      €0–€60 per year |                                                                        1 |                €0–€60 |
| Durable checkpoint identification labels                           |                   €2 |                                                                    20–24 | Approximately €40–€48 |
| Optional charging station or secure device cabinet                 |             €80–€200 |                                                                        1 |              €80–€200 |

The hardware cost for a typical pilot site would therefore be approximately:

```text
€660 to €880
```

This amount depends mainly on whether the client already owns compatible Android devices.

### Recurring Monthly Costs for a Pilot Site

| Item                                                         |                                                                                            Indicative Cost |
| ------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------: |
| Hosting, including a managed database and application server |                                                                                         €40–€100 per month |
| Domain name and certificate                                  |                                                                                 Approximately €2 per month |
| Corrective and evolutionary maintenance                      | To be defined according to the required service-level agreement, for example one maintenance day per month |
| Monitoring, logs and infrastructure alerts                   |                                       €0–€30 per month, because free tools may be sufficient at this scale |
| Automated backup storage                                     |                                                                             Approximately €5–€20 per month |
| Optional mobile-device management                            |                                         Depends on the number of managed devices and the selected provider |

### Development Estimate for a Production Project

This estimate applies to a real pilot deployment beyond the one-week proof of concept.

| Work Package                            | Scope                                                                                                                                   |                 Estimated Effort |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------: |
| **Sprint 0 — Discovery and Definition** | Client workshops, clarification of operational rules, patrol frequency, roles, thresholds and escalation procedures                     |                    3 person-days |
| **Sprint 1 — MVP**                      | NFC scanning, authentication, real-time dashboard and basic history—the MUST HAVE features                                              |                   10 person-days |
| **Sprint 2 — Operational Features**     | CRUD management for secured areas and NFC patches, detailed roles, configurable thresholds and offline support—the SHOULD HAVE features |                    8 person-days |
| **Sprint 3 — Security Hardening**       | Account locking, audit logging, load testing, TLS configuration and pilot deployment                                                    |                    6 person-days |
| **Pilot-Site Acceptance and Training**  | Real patrol tests with security guards, corrections and user training                                                                   |                    4 person-days |
| **Total**                               |                                                                                                                                         | **Approximately 31 person-days** |

With an indicative daily rate of **€500 per person-day**, the estimated development budget is:

```text
31 person-days × €500 = €15,500
```

The indicative development budget for the pilot site is therefore approximately:

# **€15,500 excluding hardware and recurring costs**

The final price must be refined with the client according to:

* the actual daily rate of the service provider;
* the exact number of secured checkpoints;
* the number of user roles;
* the required security and audit level;
* the number of developers working in parallel;
* the expected support and maintenance agreement;
* integration requirements with existing security systems;
* the number of secured sites included in the initial deployment.

The schedule below assumes that two developers work full-time on the project.

## 3. Production Project Schedule

```text
Week 1:
Sprint 0 — Client workshops, requirements clarification and definition
of checkpoint thresholds, patrol rules and user roles.

Weeks 2–3:
Sprint 1 — MVP development: NFC scanning, authentication,
basic history and real-time security dashboard.

Weeks 4–5:
Sprint 2 — Secured-area and NFC-patch administration,
detailed permissions, configurable thresholds and offline support.

Week 6:
Sprint 3 — Security hardening, account protection,
audit logging, infrastructure configuration and load testing.

Week 7:
User acceptance testing at the pilot secured site,
complete patrol tests and security-guard training.

Week 8:
Pilot go-live with enhanced monitoring and support.

Week 9 onward:
Progressive deployment to additional secured sites,
according to the number of locations and the client’s rollout priorities.
```

### Visual Project Timeline

| Phase                      | Week 1 | Week 2 | Week 3 | Week 4 | Week 5 | Week 6 | Week 7 | Week 8 | Week 9+ |
| -------------------------- | -----: | -----: | -----: | -----: | -----: | -----: | -----: | -----: | ------: |
| Discovery and requirements |      ■ |        |        |        |        |        |        |        |         |
| MVP development            |        |      ■ |      ■ |        |        |        |        |        |         |
| Operational features       |        |        |        |      ■ |      ■ |        |        |        |         |
| Security hardening         |        |        |        |        |        |      ■ |        |        |         |
| Acceptance and training    |        |        |        |        |        |        |      ■ |        |         |
| Pilot go-live              |        |        |        |        |        |        |        |      ■ |         |
| Multi-site rollout         |        |        |        |        |        |        |        |        |       ■ |

## 4. Pilot-Site Deployment Approach

The pilot deployment should take place on one secured site with a limited and clearly defined perimeter.

The recommended pilot scope includes:

* one main security entrance or control station;
* one security supervisor;
* two or three security guards;
* approximately 10 to 20 NFC checkpoints;
* one complete patrol route;
* several days of enhanced monitoring after go-live.

Before activation, each checkpoint must be:

1. physically inspected;
2. assigned a unique NFC patch;
3. registered in the administration interface;
4. associated with a secured area;
5. tested using the target Android device;
6. labelled so that the guard can identify the checkpoint;
7. included in the official patrol procedure.

During the pilot period, supervisors should monitor:

* missed checkpoints;
* delayed patrols;
* damaged NFC patches;
* failed login attempts;
* offline synchronisations;
* unusual differences between `scanned_at` and `received_at`;
* feedback from security guards;
* false alerts caused by incorrectly configured thresholds.

## 5. Training and Change Management

The solution is designed to remain simple enough for security personnel to use with minimal training.

A typical training session would last approximately 30 to 45 minutes and cover:

* logging in with a badge identifier and PIN;
* activating the NFC reader;
* scanning a checkpoint;
* recognising successful and failed scans;
* understanding offline mode;
* checking pending synchronisations;
* reporting a damaged or missing NFC patch;
* logging out at the end of a shift;
* contacting the security supervisor in the event of a technical problem.

The security supervisor would receive additional training covering:

* real-time supervision;
* alert interpretation;
* history consultation;
* NFC-patch registration;
* checkpoint configuration;
* account management;
* incident reporting;
* audit-log consultation.

A short user guide and an operational fallback procedure should be provided.

## 6. POC Reuse and Project Acceleration

The proof of concept delivered during the current one-week project already covers the technical foundations of the proposed production system.

It includes:

* user authentication;
* role-based access;
* NFC checkpoint scanning;
* real-time WebSocket supervision;
* checkpoint history;
* NFC-patch administration;
* offline scan storage;
* automatic synchronisation;
* configurable alert states;
* a functional security dashboard.

The POC therefore already covers the technical scope of Sprint 1 and a significant part of Sprint 2.

The proposed production schedule does not start from zero. It starts from an existing functional foundation that must be tested, secured, documented and adapted to the operational requirements of the pilot secured site.

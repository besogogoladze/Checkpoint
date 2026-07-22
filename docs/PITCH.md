# Company Pitch Framework — To Be Completed by the Team

This document cannot be completed entirely on your behalf. The credibility of a company pitch must come from verifiable facts about your own team, not from generic statements.

The structure below reflects what the client expects and explains what makes each section convincing—or what may make it sound artificial or unprepared.

## 1. Who Are We?

Complete this section with:

* **The name of your company or studio**, and the reason behind that name. A simple but meaningful explanation is more memorable than a generic name chosen five minutes before the presentation.

* **The company creation date.** Even for a bootcamp project, choose a realistic date that matches the actual formation of the team.

  Do not claim ten years of experience if the jury knows that the company was created during the bootcamp. A young, specialised and honest team is more credible than a fictional company history that cannot withstand a simple question.

* **The size of the team** and the responsibilities of each member.

A possible introduction could follow this structure:

```text
We are [Company Name], a young digital solutions studio created in [Month and Year].

We specialise in secure web applications, real-time supervision systems and
connected technologies such as NFC.

Our team consists of [number] members with complementary skills in backend
development, frontend development, security, testing and project management.
```

The final version must reflect the team’s real situation.

## 2. Who Are Our Developers?

Present each team member with:

* their actual role on this project;
* their main technical responsibilities;
* the features they personally delivered;
* the technologies they used;
* the part of the demonstration they can explain in detail.

Avoid empty or exaggerated titles.

For a team of three or four people, titles such as the following are more credible:

* NFC integration lead;
* real-time dashboard developer;
* backend and database developer;
* authentication and security lead;
* testing and quality coordinator;
* frontend and user-experience developer.

These roles are more meaningful than assigning titles such as “CTO” or “Chief Architect” without corresponding experience or responsibilities.

A team-member presentation can use this format:

```text
[Name] — NFC and Mobile Integration

Responsible for Web NFC integration, Android compatibility testing and
offline scan management.

Delivered:
- NFC reader activation;
- checkpoint identification;
- local offline queue;
- automatic synchronisation after network recovery.
```

Every technical claim should be consistent with the Git history.

The jury may ask targeted questions about a specific feature. The person who implemented it should be able to explain:

* how it works;
* why that approach was selected;
* what problems were encountered;
* how those problems were resolved;
* what could be improved in a production version.

### Important Risk to Avoid

Do not present one “star developer” as the person who completed the entire project.

A team where only one person understands the application creates a major operational risk.

From the client’s perspective, this is a single point of failure. If that person becomes unavailable, maintenance, support and future development may stop.

A credible company demonstrates that:

* responsibilities were distributed;
* features were reviewed by another team member;
* important technical knowledge was shared;
* documentation was produced;
* several people can maintain the critical parts of the system.

## 3. What Are Our References?

For a bootcamp or training project, there are two honest approaches.

### Option 1 — Present a Real Previous Team Project

When the team has already delivered another project together, present it truthfully.

Explain:

* the project objective;
* the technologies used;
* the main difficulty;
* the result achieved;
* what the team learned;
* what was reused in the NFC patrol project.

Example:

```text
During our previous team project, we developed a real-time order management
system.

That experience taught us to define user roles early, separate business logic
from the interface and test real-time events between multiple user profiles.

We reused these lessons when designing the security supervision dashboard.
```

Do not invent a commercial deployment.

For example, do not claim:

```text
We deployed this system across three secured sites.
```

when the actual project is a one-week proof of concept.

The jury understands the context. A fictional reference can damage credibility more than openly presenting a young company with a functional product.

### Option 2 — Present the Current POC as a Reference

The current proof of concept is already a strong reference.

A clear statement could be:

```text
Our strongest reference is the solution we are presenting today.

In one week, our team designed and delivered a functional NFC security patrol
system with authentication, checkpoint scanning, offline support, history and
real-time supervision.

The NFC scan demonstrated in front of you is not a simulation. It is a working
technical proof of our ability to deliver.
```

A real NFC tag scanned during the presentation provides stronger evidence than several invented references.

## 4. How Do We Work?

This section should use the delivery method, tools and organisation described in `OFFRE.md`, but it should be presented from the client’s point of view.

Avoid simply saying:

```text
We used Scrum.
```

Instead, explain how the chosen method reduced project risk.

For example:

```text
We identified NFC compatibility as the main technical risk, so we tested it
at the beginning of the project rather than leaving it until the final day.

We divided the work between team members to avoid relying on a single
developer.

Each major feature was reviewed before integration.

When the client requirement was unclear, we documented the open question and
clarified it before implementing assumptions that could later become expensive
to correct.
```

The team should explain how it handled:

* NFC compatibility risks;
* Android device testing;
* role and permission design;
* real-time dashboard updates;
* unstable network conditions;
* offline scan storage;
* security requirements;
* team coordination;
* Git branches and pull requests;
* testing and acceptance criteria;
* changes in project scope.

### Example Working Method

```text
We work in short iterations with a clear priority: validate the highest-risk
feature first.

For this project, we started with real NFC reading on an Android device.

Once NFC compatibility was confirmed, we connected the scan to the backend,
stored the checkpoint validation and displayed it on the supervision dashboard
through WebSocket communication.

The team worked on separate responsibilities but reviewed each other’s work
before merging it into the main branch.

This approach allowed us to detect technical problems early and maintain a
working version of the application throughout the project.
```

## 5. The “Wow” Effect

The strongest “wow” effect is not a complicated animation, an exaggerated speech or an unrealistic promise.

The most convincing demonstration is:

1. a real NFC tag installed as a security checkpoint;
2. a security guard scanning it with an Android smartphone;
3. the scan being sent to the server;
4. the security dashboard updating in less than one second;
5. the checkpoint changing status;
6. the scan appearing in the patrol history;
7. the team explaining exactly how the complete flow works.

The demonstration scenario could be:

```text
A security guard begins a patrol round.

The guard reaches a secured checkpoint and scans the NFC tag.

The application identifies the checkpoint, records the guard, date and time,
and sends the information securely to the server.

At the same moment, the security control station receives the update through
WebSocket communication.

The supervisor can immediately see that the checkpoint has been validated.
```

The presentation becomes even stronger when the team also demonstrates a failure scenario, such as:

* scanning an unknown NFC tag;
* losing the network connection;
* storing a scan offline;
* restoring the connection;
* automatically synchronising the pending scan;
* displaying a damaged checkpoint;
* showing an overdue checkpoint in red.

## 6. Recommended Pitch Structure

A concise company pitch can follow this order.

### Opening

```text
Good morning.

We are [Company Name], a young development team specialising in secure,
real-time and connected web applications.

Today, we are presenting a solution designed to provide reliable evidence of
security patrols through NFC checkpoints.
```

### Client Problem

```text
In a secured site, supervisors need to know whether patrol rounds have actually
been completed, when each checkpoint was visited and which security guard
performed the control.

Traditional paper logs are slow, difficult to verify and do not provide
real-time visibility.
```

### Proposed Solution

```text
Our solution allows a security guard to scan NFC checkpoints during a patrol
using an Android smartphone.

Every scan is authenticated, recorded and immediately displayed on the
security supervision dashboard.

The system also supports offline operation when the network is temporarily
unavailable.
```

### Team Credibility

```text
Our team divided the project into clear technical responsibilities:
NFC integration, backend and database, real-time dashboard, security and
testing.

Each critical feature was reviewed and tested by more than one team member.
```

### Delivery Evidence

```text
Rather than only describing what we could build, we built it.

The proof of concept presented today includes real NFC scanning,
authentication, checkpoint administration, offline synchronisation,
history and real-time supervision.
```

### Live Demonstration

```text
We will now demonstrate the complete patrol flow using a real NFC checkpoint.
```

### Closing

```text
Our objective is simple: provide security supervisors with immediate,
reliable and traceable evidence that every required checkpoint has been
controlled.

We are not presenting only an idea. We are presenting a functional solution
that can be tested today and prepared for deployment on a real secured site.
```

## 7. Final Credibility Checklist

Before the presentation, the team should be able to answer the following questions:

* Why did you choose NFC instead of QR codes?
* Why did you choose Web NFC instead of a native Android application?
* What happens when the network is unavailable?
* How do you prevent an unauthorised person from recording a patrol?
* How is a checkpoint associated with an NFC patch?
* What happens when a patch is damaged?
* How quickly does the dashboard update?
* How are passwords or PIN codes protected?
* How are user roles managed?
* Who implemented each feature?
* What did each team member contribute?
* What would need to change before production deployment?
* How would the architecture support multiple secured sites?
* What is the estimated production cost?
* What is the project deployment schedule?

A credible team does not need to pretend that the POC is already a finished industrial product.

It must clearly explain:

* what currently works;
* what has been demonstrated;
* what remains to be strengthened;
* how the solution could move from POC to production;
* why the team can deliver that next stage.

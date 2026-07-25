# 🛠️ TryHackMe: ToolsRUs — Full Architectural & Deep-Dive Analysis

This documentation serves as a permanent reference architecture for attacking poorly managed Apache Tomcat assets without causing service disruption.

---

## 🎯 Target Overview
* **Target Operating System:** Linux (Ubuntu)
* **Application Frame:** Apache HTTP Server & Apache Tomcat Container
* **Network Protocol Connector:** Apache-Coyote/1.1
* **Objective:** Secure an interactive terminal and read the system root flag (`/root/flag.txt`)

---

## 🧭 Operational Timeline (The Attack Chain)


### 1. Perimeter Enumeration (Reconnaissance)
Mapped the host interfaces using network-layer discovery to identify active service endpoints.
* **Core Action:** Validated distinct daemon layouts running on standard and high-port allocations.
* **Findings:** Identified standard Apache HTTP infrastructure alongside an isolated Apache Tomcat deployment.

### 2. Directory Brute-Forcing (Content Discovery)
Audited the primary web port to discover internal documents, configurations, or hidden administrative directories.
* **Core Action:** Discovered active directory endpoints using a targeted alphanumeric common wordlist.
* **Key Discovery Paths:**
  * `/guidelines/` — An unprotected staging location exposing internal system documentation. Reviewing this document explicitly leaked a valid operational username: **`bob`**.
  * `/protected/` — A locked asset throwing an `HTTP Status: 401 Unauthorized` block, strictly requiring browser-native authentication parameters.

### 3. Identity Cracking (HTTP Basic Authentication)
Targeted the explicit `/protected/` URI using the extracted identity profile to reverse-engineer access parameters.
* **Core Action:** Executed high-thread authentication attempts passing the leaked username against a standard alphanumeric wordlist.
* **Result:** Successfully cracked Bob's network credential mapping: **`bob:bubbles`**.

### 4. Cross-Service Pivot (Tomcat Auditing)
Tested the recovered credential set against the administrative management interfaces of Apache Tomcat located on the secondary high-port allocation.
* **Core Action:** Validated credential reuse vulnerabilities. Bob re-used his entry token on the primary web tier and the core application cluster dashboard.
* **Technical Footprint:** Executed precise application layer headers to verify that the active server routing was handled by **`Apache-Coyote/1.1`**, while finding **`5` separate misconfigured default documentation artifacts** left active inside the target root directory.

### 5. Weaponization & Exploit Modification
Initial deployment automation attempts utilizing standard framework modules failed with an explicit `HTTP 403 Forbidden` error because Bob's profile was restricted from interacting with the programmatic script deployment API endpoint (`/manager/text`).
* **The Strategic Engineering Correction:** Shifted to an interactive web interface injection payload module (`tomcat_mgr_upload`).
* **The Mechanics:** The attack module initiated a stateful session facsimile, handled the initial authentication sequence, dynamically parsed the server's backend parameters to extract a valid, time-sensitive **CSRF (Cross-Site Request Forgery) Token**, and executed a multi-part HTML upload payload container.
* **Payload Archetype:** Injected a Java Web Archive (`.war`) container encapsulating a Java-based reverse TCP stager. The host successfully deployed and unpacked the container, initializing an interactive **Meterpreter session** back to the active listening terminal.

### 6. Interactive Post-Exploitation & Data Triage
Dropped from the interactive framework stager directly into a native system shell to inspect current process ownership and complete the target directive.
* **Operational Privileges:** Running `whoami` immediately answered **`root`** without any additional host-level privilege escalation required.
* **Data Exfiltration:** Navigated directly to the primary root volume space and unpacked the protected flag file via absolute path indexing:
  ```bash
  cat /root/flag.txt
  ```

---

## 🧠 Deep-Dive Security Lessons (Core Principles)

### 🔑 Principle 1: The Danger of Credential Reuse Vectors
System administrators often treat application layer boundaries with varying tiers of security attention. Deploying an identical credential profile (`bob:bubbles`) across an unprivileged web repository and a critical container manager allows an attacker to leverage minor info leaks to compromise full internal backend clusters.

### 🛡️ Principle 2: Role-Based Access Controls (RBAC) & CSRF Defense Mechanics
Modern application layers utilize unique CSRF security tokens inside their HTML Graphical User Interfaces to prevent automated scripts from conducting blind executions. When programmatic text APIs (`manager-script`) throw strict `403 Forbidden` access rejections, the defense can be natively bypassed by using exploitation modules that mimic authentic browser sessions and drop components directly into active HTML form file uploads.

### 🛑 Principle 3: Process Isolation & The Danger of Root Daemons
Production application daemons should never be launched directly out of high-privileged system spaces. They must be bound exclusively to low-privilege service accounts (e.g., `www-data` or `tomcat`). Because the system administrator improperly configured the core Apache Tomcat container binary to run as the native host **`root`** user, any remote code execution achieved within the application instantly inherited absolute kernel dominance.

# Project 1: DDoS Mitigation Tool

## Overview
Students will build a **network monitoring system** that detects Distributed Denial of Service (DDoS) attacks through traffic analysis, establishes baseline behavior patterns, and automatically implements rate limiting or firewall rules to mitigate attack impact.  

This project teaches **network security**, **anomaly detection**, and demonstrates **defensive techniques** used by infrastructure providers and security operations centers.

---

## Step-by-Step Instructions

### 1. Learn DDoS Attack Types
- **Volumetric attacks**: Flood target with massive traffic volume.  
- **Protocol attacks**: Exploit weaknesses in protocols (e.g., SYN floods).  
- **Application-layer attacks**: Target specific services with seemingly legitimate requests.  

**Detection approaches:**  
- Volumetric → obvious traffic spikes  
- Protocol → unusual packet patterns  
- Application → suspicious request rates  

---

### 2. Establish Traffic Baseline
- Use **[Scapy](https://scapy.readthedocs.io/en/latest/)** for packet sniffing.  
- Collect statistics: packet counts per protocol, source/destination IP distributions, port usage, bandwidth trends.  
- Store baseline metrics over days/weeks to capture diurnal patterns.  
- Build statistical models (mean, std. deviation, percentiles) for anomaly detection.

---

### 3. Real-Time Traffic Monitoring
- Continuously capture and analyze live packets.  
- Compute traffic statistics per time window (e.g., per minute).  
- Compare against baseline patterns.  
- Optimize performance: rolling windows, incremental aggregation, sampling.

---

### 4. Implement Anomaly Detection
- Flag deviations (e.g., traffic exceeds baseline mean by 5σ).  
- Indicators:  
  - Sudden traffic spikes  
  - Protocol distribution shifts  
  - Source IP concentration  
  - Port concentration  
- Combine signals into a composite anomaly score.

---

### 5. Automated Mitigation
- **Rate limiting**: Token bucket algorithm.  
- **Firewall rules**: Use `iptables` to block malicious sources.  
- **Safeguards**:  
  - Lenient rate limiting first  
  - Gradual restrictions  
  - Whitelist critical IPs  

---

### 6. Alerting & Escalation
- Notify admins via **email, Slack, WhatsApp, or system notifications**.  
- Include details: attack type, affected services, mitigation status, source IPs/countries.  
- Implement **alert ranking** to avoid notification fatigue.

---

### 7. Monitoring Dashboards
- Show real-time traffic stats, attack indicators, active mitigations, historical trends.  
- Visualize:  
  - Bandwidth usage  
  - Packet rates  
  - Protocol distribution  
  - Geographic attack sources  
- Provide admin controls: adjust rate limits, whitelist/blacklist IPs, trigger mitigations.

---

### 8. Documentation
- Explain DDoS concepts, attack types, mitigation strategies.  
- Provide deployment guidance and performance considerations.  
- Discuss limitations (e.g., massive attacks may require ISP-level mitigation).  
- Compare with commercial DDoS protection services.  
- Include incident response workflows and stakeholder communication templates.

---

## 📊 Suggested Dataset
Students can start with **publicly available datasets**:
- **CAIDA DDoS Attack Dataset** (Center for Applied Internet Data Analysis)  
- **UNSW-NB15 Dataset** (network intrusion dataset with DDoS samples)  
- **CICDDoS2019 Dataset** (Canadian Institute for Cybersecurity, labeled DDoS traffic)  

These datasets provide labeled attack vs. normal traffic, ideal for testing anomaly detection algorithms.

---

## 🛠 Recommended Libraries & Tools
- **Scapy** → packet sniffing & traffic analysis  
- **Pandas / NumPy** → statistical modeling & anomaly detection  
- **Matplotlib / Seaborn / Plotly** → dashboards & visualization  
- **iptables / ufw** → firewall rule automation  
- **Flask / Django** → web-based monitoring dashboard  
- **Celery / APScheduler** → background monitoring & alerting tasks  
- **Slack API / Twilio / SMTP** → alert notifications  

---

## 🎯 Deliverables
- Baseline traffic pattern establishment  
- Real-time traffic monitoring system  
- Anomaly detection algorithms  
- Automated rate limiting implementation  
- Firewall rule generation & management  
- Alert system with escalation  
- Admin dashboards and controls  

# Project 2: SBOM Generator

## Overview
Students will build a **Software Bill of Materials (SBOM) generator** that scans project directories, identifies dependencies across multiple ecosystems, resolves transitive dependency trees, and produces SBOM documents in both **[SPDX](https://spdx.dev/)** and **[CycloneDX](https://cyclonedx.org/)** formats.  

The tool will also cross-reference discovered packages against CVE databases (NVD, OSV) to generate vulnerability reports alongside the SBOM.  

This project is directly relevant to **[US Executive Order 14028](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/)** on improving the nation's cybersecurity, which mandates SBOM production for all software sold to the federal government. Mastering SBOM generation is therefore a **marketable real-world skill**.

---

## Step-by-Step Instructions

### 1. Set Up Project & Parser Framework
- Create a module with a **plugin-style architecture**.  
- Each ecosystem has its own parser implementing a common `DependencyParser` interface:  
  - `Detect(dir string) bool`  
  - `Parse(dir string) ([]Package, error)`  
- Define shared types:  
  - `Package` → name, version, ecosystem, license, checksums  
  - `DependencyTree` → package + resolved transitive dependencies  
  - `Vulnerability` → CVE ID, severity, affected versions, fix version  
- Build a CLI with subcommands for scanning, generating SBOMs, and checking vulnerabilities.

---

### 2. Build Dependency Parsers
- Implement parsers for **Python** (`requirements.txt`, `pyproject.toml`, `Pipfile.lock`, `uv.lock`) and **Java** (`pom.xml`, `build.gradle`).  
- Extract direct dependencies with version constraints from manifest files.  
- Use **lockfiles** (Python and Java both support them) to capture exact resolved versions.  
- Handle version constraint syntax: semver ranges, tilde, caret, wildcards.  
- Prioritize lockfile data over manifest data.

---

### 3. Transitive Dependency Resolution
- For ecosystems with lockfiles: parse lockfiles directly.  
- For ecosystems without lockfiles: query package registry APIs.  
  - Python → PyPI JSON API  
- Build a **dependency graph**:  
  - Track direct vs transitive relationships  
  - Depth level  
  - Multiple versions (conflicts)  
- Implement **cycle detection** for circular dependencies.

---

### 4. License Detection & Metadata Enrichment
- Detect licenses via:  
  - Lockfiles (if included)  
  - Registry APIs  
  - Local source files (`LICENSE`, `COPYING`, SPDX headers)  
- Map license strings to **SPDX identifiers**.  
- Enrich metadata: download URL, homepage, maintainer info, SHA-256 checksums.  
- Flag:  
  - Missing licenses  
  - Copyleft licenses in proprietary projects  
  - Problematic licenses  

---

### 5. Generate SPDX SBOMs
- Implement **SPDX 2.3** generation in JSON and tag-value formats.  
- Map internal structures to SPDX concepts:  
  - Each package → `SPDXPackage`  
  - Relationships → `DEPENDS_ON`, `BUILD_TOOL_OF`, etc.  
- Include document metadata: creator tool, timestamp, namespace, CC0-1.0 license.  
- Validate against **SPDX JSON schema**.

---

### 6. Generate CycloneDX SBOMs
- Implement **CycloneDX 1.5** generation in JSON.  
- Map packages to `component` objects: type, name, version, purl, licenses, hashes.  
- Represent dependency tree with `dependencies` objects.  
- Include metadata: tool identification, manufacture date.  
- Validate against **CycloneDX JSON schema**.  
- Support both flat and hierarchical dependency views.

---

### 7. Vulnerability Scanner
- Integrate with:  
  - **NVD API** (CPE identifiers)  
  - **OSV API** (Package URLs)  
- For each package:  
  - Query vulnerabilities  
  - Record CVE ID, CVSS score, severity, affected versions, fix version, description  
- Generate vulnerability report sorted by severity.  
- Optionally produce **VEX (Vulnerability Exploitability Exchange)** documents.

---

### 8. CI/CD Integration & Policy Enforcement
- Add `--check` mode for policy evaluation.  
- Policies:  
  - No critical CVEs  
  - No unknown licenses  
  - No dependencies older than threshold  
  - Max allowed dependency depth  
- Provide GitHub Actions & GitLab CI templates.  
- Block merges if violations occur.  
- Write integration tests with known fixtures.

---

## 🔑 Key Concepts to Learn
- Purpose of SBOMs & regulatory requirements (**EO 14028**)  
- **SPDX vs CycloneDX** specification structures  
- Dependency resolution across ecosystems  
- Transitive dependency trees & supply chain depth  
- License compliance & SPDX identifiers  
- CVE databases (NVD, OSV) & vulnerability matching  
- VEX documents for vulnerability communication  
- CI/CD integration for supply chain security  

---

## 🎯 Deliverables
- CLI tool with pluggable ecosystem parsers  
- Parsers for Python and Java  
- Transitive dependency resolution with graph visualization  
- SPDX 2.3 and CycloneDX 1.5 SBOM generation with schema validation  
- Vulnerability scanner integrating NVD & OSV  
- Policy enforcement engine with configurable rules  
- CI/CD templates (GitHub Actions, GitLab CI)  
- Integration tests with project fixtures  

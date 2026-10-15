
# FORGE-RFDT

## Forensic Reproducibility and Grounded Experimentation — Radio Frequency & Digital Twin

**FORGE-RFDT** (**Forensic Reproducibility and Grounded Experimentation — Radio Frequency & Digital Twin**) extends the FORGE virtualized IT/OT experimental baseline toward **physical, virtual and hybrid industrial infrastructures**, **digital-twin-based observation**, and **radio-frequency (RF) forensic evidence acquisition and analysis**. The platform's distinctive goal is to investigate an incident across computing systems, industrial control and physical processes, wireless transmissions and their digital representations without treating model outputs as primary evidence.

The project brings together the established FORGE case-centered DFIR workflow, the industrial 3D and signal-mapping capabilities of [TwinAxisLab](https://github.com/Younyat/TwinAxisLab), the digital-twin divergence perspective of [FORGE-DT](https://github.com/Younyat/FORGE-DT), and a proposed RF acquisition and analysis layer inspired by the separate [SpectraRFX](https://github.com/Younyat/SpectraRFX) project.

> **Implementation boundary:** the repository inherits documented FORGE capabilities. Physical-machine integration, TwinAxisLab/FORGE-DT interoperability and the new RF evidence pipeline are described as **integration targets** unless their code, device tests and evidence outputs have been verified in this repository. No RF decoder, continuous RF capture, synchronized multi-receiver acquisition or RF causal attribution is asserted to be operational solely because it appears in this roadmap.

Three infrastructure modes are in scope: **virtual** (OpenStack, simulated PLCs and services), **real** (authorized PLCs, robots, sensors, gateways and RF receivers), and **hybrid** (physical and virtual components working together). The existing virtual workflow remains supported; OpenStack is not a prerequisite for every physical-equipment or RF acquisition scenario.

The workflow encompasses incident execution, detection, memory-/volatility-aware acquisition, primary-evidence preservation, post-acquisition analysis, 3D and digital-twin inspection, causal reconstruction and comparisons across repeated experiments. Research and training are the principal intended contexts.

```text
Scenario definition (virtual / physical / hybrid)
  -> prepare hosts, PLCs, industrial assets, digital twins and RF sensors
  -> establish time sources, capture bands/channels and provenance
  -> monitor traffic, machine states, twin observations and RF environment
  -> controlled incident / fault scenario (only where authorized and safe)
  -> detect and contextualize incident
  -> acquire digital, OT, physical telemetry and RF evidence
  -> preserve originals, hashes, manifests and chain-of-custody events
  -> run offline RF decoding, spectrum analysis and multi-layer analysis
  -> reconstruct timeline / evidence graph / 3D replay
  -> compare observations with digital-twin expectations
  -> report supported, ambiguous and missing causal relations
```

The next section describes the infrastructure required to run this workflow.

---

## Platform Evolution: Virtual, Real, Hybrid and RF-Aware IT/OT

FORGE-RFDT **does not replace or remove** the virtual-machine-based capabilities of FORGE-VI. It broadens the supported *scenario model* to include:

| Scenario mode | Infrastructure | Intended observations |
|---|---|---|
| **Virtual** | OpenStack nodes, virtual PLC, virtual SCADA/HMI and simulated equipment | PCAP, memory, snapshots, alerts, simulated OT state |
| **Physical** | Real PLCs, robots, sensors, actuators, industrial PCs and IoT gateways | Network traces, available controller logs/registers, sensor values, machine state and gateway records |
| **Hybrid** | Real equipment interacting with virtual hosts, emulated PLCs or 3D twins | Correlated physical and virtual events, command paths, state changes and detection results |

**Implementation status:** the README below documents an existing OpenStack-centered DFIR baseline. The real-asset onboarding, TwinAxisLab integration and FORGE-DT observability discussed here are integration objectives, not a declaration that the current FORGE-RFDT repository already implements or validates every capability.

### Integration with TwinAxisLab

The planned industrial 3D integration will draw on [TwinAxisLab](https://github.com/Younyat/TwinAxisLab). Its independently documented capabilities include asset import, mechanical/kinematic models, functional components, industrial signal mapping, live twin monitoring, historical replay and connectivity through Modbus, MQTT, and an OPC UA gateway. FORGE-RFDT should use these as a **separate engineering and operational layer**, not silently treat 3D geometry or its runtime as original forensic evidence.

The integration target includes:

- import of 3D industrial assets and configurable joint/axis models;
- representation of physical equipment through inspectable digital counterparts;
- explicit signal bindings for industrial controller variables and sensor telemetry;
- observation of *reported* versus *expected* or *desired* state, with source and quality metadata;
- safe read-only inspection by default and separately authorized, controlled command pathways;
- collection of event and trace records as potential input to the FORGE-RFDT preservation pipeline;
- replay of historical state **without replay-triggered commands to physical equipment**.

### Integration with FORGE-DT

[FORGE-DT](https://github.com/Younyat/FORGE-DT) is a related branch of the FORGE family. The intended integration brings digital-twin-oriented analysis into the case-centered workflow, especially:

- explicit expected, commanded, observed and reconstructed state models;
- investigation of state divergence under controlled fault or incident scenarios;
- evidence-backed links between controller commands, telemetry, physical response and twin state;
- structured uncertainty where real-world visibility is incomplete;
- separation between forensic observations, modeled expectations, and derived conclusions.

These digital-twin capabilities must be integrated and evaluated within FORGE-RFDT before they can be claimed as operational features of this repository.

### Safety and scientific boundary

A physical industrial process is **not** equivalent to a disposable virtual test bed. Any active probing, fault injection, write command, robot motion, PLC modification or attack scenario requires explicit authorization, device-specific interlocks, documented safe ranges, emergency-stop arrangements and an appropriate test environment. Baseline virtual attack automation must not be automatically enabled on production systems.

---

## 1. Infrastructure Deployment

The **legacy virtualized deployment mode** uses a local OpenStack environment. FORGE-RFDT additionally targets real industrial assets and mixed physical–virtual scenarios; these modes do **not** inherently require that physical PLCs, robots, or gateways run inside OpenStack. Physical connectivity, credentials, network segmentation, safety interlocks, and compatible acquisition backends must be configured separately. The commands below describe the retained OpenStack workflow, not a verified universal installer for physical integration.

### 1.1 Baseline Host Requirements

Use the following baseline for a stable deployment:

- **Ubuntu 24.04 LTS**
- **8 CPU cores**
- **48 GB RAM**
- **500 GB of free disk space**
- **Hardware virtualization enabled**

If the platform is executed inside **VirtualBox** or **VMware**, virtualization must be enabled in the BIOS or UEFI and exposed to the guest operating system. In practice, this requires **nested virtualization**.

Without nested virtualization, the OpenStack environment may fail to deploy correctly or may behave unreliably.

The historical OpenStack baseline reports approximately **30 minutes** for deployment under the stated conditions; this is not a measured deployment time for real or hybrid installations.

### 1.2 Deploy the OpenStack Environment

Run the installer from the project root:

```bash
bash openstack-installer/openstack-installer.sh
```

After deployment, the OpenStack virtual environment is created at:

```bash
openstack-installer/openstack_venv
```

The OpenStack credentials file is generated at:

```bash
admin-openrc.sh
```

For public deployments or shared repositories, use a safe template such as:

```bash
admin-openrc.example.sh
```

and avoid committing environment-specific credentials.

### 1.3 Start the Platform UI

To launch the FORGE-RFDT dashboards, run:

```bash
bash start_dashboard.sh
```

This script is located in the project root.

On the first launch, startup may take longer because dependencies may need to be installed.

### 1.4 Recover OpenStack Services After Disk-Related Failures

If OpenStack services stop because the host has run out of disk space, first recover free space and then restart the services with:

```bash
bash restart_openstack.sh
```

This script is also located in the project root.

After the infrastructure is available, the platform workflow can be executed from deployment to reconstruction.

---

## 2. Platform Workflow

FORGE-RFDT retains the original virtual DFIR workflow and extends its target architecture toward physical and hybrid scenarios. The existing sequence is:

1. **Deploy the OpenStack infrastructure**
2. **Start the platform dashboards**
3. **Create the base IT scenario**
4. **Extend the scenario with industrial components when needed** (virtual devices in the existing editor; real-device onboarding requires the corresponding connector)
5. **Install the required tools on the deployed nodes**
6. **Access the installed tools through the operational portal**
7. **Execute controlled attack and detection exercises**
8. **Trigger forensic escalation when incident conditions require acquisition**
9. **Acquire and preserve evidence through a profile-conditioned workflow**
10. **Review the preserved case, manifest, chain of custody, and pipeline events**
11. **Run post-acquisition analysis over preserved artifacts**
12. **Reconstruct the incident through the FOC layer**
13. **Compare repeated executions through reproducibility-oriented views**

This workflow allows the user to move from infrastructure provisioning to case-centered forensic reconstruction without leaving the platform.

The next section explains the methodological model behind the workflow.

---

### Real and Hybrid Workflow Extension

When the corresponding integrations are available, an experimental run should additionally: (1) register authorized physical assets, (2) establish read-only monitoring and map signals to a twin, (3) synchronize available clocks and record uncertainties, (4) preserve command, telemetry and physical-state observations, (5) compare expected and observed state, and (6) reconstruct the incident with a clear distinction between observations and model-derived explanations. OpenStack provisioning remains optional for a purely physical scenario.

## 3. Scientific Workflow Model

FORGE-RFDT is not designed as a collection of isolated tools. It preserves the controlled DFIR methodology of the virtualized IT/OT baseline while broadening the *target scope* to real industrial systems and their digital counterparts. Actual coverage of physical sources depends on device access, supported protocols, time synchronization and acquisition readiness.

The platform follows a simple methodological rule:

```text
Preserve first.
Analyze second.
Reconstruct third.
Compare cautiously.
```

This rule means that:

- primary evidence is acquired and preserved before high-level interpretation;
- derived reports never replace preserved artifacts;
- reconstruction is performed over preserved and derived evidence records, not over live systems alone;
- uncertainty, degradation, ambiguity, and unsupported relations remain visible;
- cross-run comparison is based on structured case outputs rather than informal observations.

The workflow is organized into four layers.

### 3.1 Controlled Intervention Layer

This layer records what scenario was declared, what topology was deployed, what tools were installed, what incident profile was executed, and what detections were expected.

It includes:

- scenario definitions;
- IT and OT node roles;
- network topology;
- attack profile metadata;
- detection expectations;
- tool and sensor preparation;
- run identifiers;
- execution records.

### 3.2 Preservation Layer

This layer turns an incident execution into a forensic case.

It includes:

- evidence acquisition;
- case directory creation;
- manifest generation;
- hash recording;
- chain-of-custody registration;
- pipeline event logging;
- integrity verification;
- metadata and timing records.

### 3.3 Analytical Layer

This layer processes preserved artifacts while keeping primary evidence separate from derived outputs.

It includes analysis over:

- evidence inventory;
- network artifacts;
- volatile memory artifacts;
- disk or snapshot artifacts;
- alert and detection records;
- OT/industrial exports;
- custody and pipeline records;
- temporal metadata;
- generated forensic reports.

### 3.4 Reconstruction and Comparison Layer

This layer evaluates what can be reconstructed from preserved and analyzed artifacts.

It includes:

- Forensic Observational Context construction;
- causal relation state assignment;
- recovered, degraded, ambiguous, and missing relation tracking;
- Causal Path Recoverability calculation;
- case-level support summaries;
- cross-run comparison profiles;
- reproducibility-oriented reporting.

The next section describes the main platform services that expose these layers to the user.

---

## 4. Main Platform Services

FORGE-RFDT is organized into a set of dashboards and services that expose the complete experimental workflow, from scenario creation to incident execution, forensic acquisition, analysis, reconstruction, and cross-run comparison.

The services are presented in the same order in which a user normally interacts with the platform.

---

### 4.1 FORGE-RFDT Home Dashboard

The **FORGE-RFDT Home Dashboard** is the main entry point of the platform.

It provides a unified overview of the current experimental environment, including scenario state, operational readiness, forensic readiness, reconstruction status, and scientific evidence indicators.

The dashboard helps the user understand whether the platform is ready for deployment, attack execution, acquisition, reconstruction, or comparison.

It exposes:

- deployed scenario status;
- available nodes and roles;
- active experiment state;
- tool and service readiness;
- attack and detection readiness;
- forensic case status;
- preservation and reconstruction indicators;
- causal relation state summaries;
- CPR-oriented reconstruction information;
- recent platform activity.

![FORGE-RFDT Home Dashboard](Images_readme/FORGE_VI_HOME_PAGE.png)

The workflow then moves from the global platform overview to scenario construction.

---

### 4.2 IT Scenario Editor

The **IT Scenario Editor** is used to create and deploy the base IT scenario on the virtualized infrastructure.

It allows the user to visually define the experimental topology by creating nodes, assigning roles, connecting them, and deploying the resulting scenario through the platform.

The editor supports:

- visual creation of IT nodes;
- role assignment, such as attacker, victim, and monitor;
- network and subnet configuration;
- image, flavor, key, and security-group selection;
- topology editing;
- scenario loading and saving;
- deployment and teardown actions.

This view defines the initial virtual environment on which tools, attacks, monitoring components, and forensic workflows are later applied.

![IT Scenario Editor](Images_readme/it_scenario_editor.png)

The IT scenario can then be extended with industrial components.

---

### 4.3 Industrial Scenario Editor

The **Industrial Scenario Editor** extends the base IT topology with OT-oriented components.

It allows the user to add industrial nodes such as PLC, SCADA, or HMI components and connect them to the existing IT scenario. In the existing implementation this creates a virtualized IT/OT environment suitable for controlled industrial cybersecurity and DFIR experimentation. The FORGE-RFDT extension additionally specifies discoverable physical assets and mixed topologies; support for individual devices must be verified rather than assumed.

The editor supports:

- loading the base IT scenario;
- adding PLC nodes;
- adding SCADA/HMI nodes;
- connecting industrial nodes to the topology;
- saving industrial extensions;
- opening deployed industrial services;
- removing industrial components when needed.

This view defines the industrial context required for OT-aware attack execution, detection, acquisition, and reconstruction.

![Industrial Scenario Editor](Images_readme/industrial_scenario_editor.png)

The deployed IT/OT environment can then be inspected through the environment dashboard.

---

### 4.4 IT/OT Environment Dashboard

The **IT/OT Environment Dashboard** provides an operational view of the deployed hybrid environment.

It helps the user inspect the current state of the virtualized IT/OT scenario and understand how the deployed nodes, services, and industrial components are organized.

It supports:

- visualization of the deployed environment;
- inspection of IT and OT roles;
- operational awareness of scenario components;
- access to node-level information;
- support for moving from deployment to tool installation and attack execution.

![IT/OT Environment Dashboard](Images_readme/it_ot_environment_dashboard.png)

Once the environment is deployed, the required tools can be installed.

---

### 4.5 Instance Tools Manager

The **Instance Tools Manager** prepares deployed nodes with the tools required for experimentation, monitoring, attack execution, detection, and forensic acquisition.

It allows the user to select a deployed instance, choose tools from a predefined catalog, and launch installation workflows with live operational feedback.

The tool catalog includes:

- security monitoring tools;
- host and network sensors;
- offensive and assessment tools;
- OT/ICS utilities;
- forensic acquisition and analysis tools;
- industrial services;
- detection and rollback profiles.

This view connects infrastructure deployment with practical experimentation by making each node operationally ready for its role.

![Instance Tools Manager](Images_readme/instance_tools_manager.png)

After tool installation, the user can access the deployed services from a role-oriented portal.

---

### 4.6 Security Training and Tools Portal

The **Security Training and Tools Portal** gives direct access to installed tools and services inside the deployed environment.

It organizes access according to scenario roles, such as attacker, monitor, victim, PLC, or SCADA/HMI. This allows the user to interact with real tools in the virtualized scenario rather than with simplified demonstrations.

The portal supports:

- role-based tool access;
- direct opening of installed services;
- access to remote instance consoles;
- operational interaction with attacker, victim, monitor, PLC, and SCADA/HMI nodes;
- hands-on training and experimentation.

![Security Training and Tools Portal](Images_readme/security_training_portal.png)

The prepared scenario can then be used for controlled attack execution.

---

### 4.7 Tactical Cyber Operations Dashboard

The **Tactical Cyber Operations Dashboard** provides the operational interface for controlled attack execution and scenario observation.

It combines target selection, attack execution, contextual node information, and operational feedback in a single interface.

The dashboard supports:

- battlefield-style scenario visualization;
- target selection;
- node intelligence;
- attack profile execution;
- attacker-side feedback;
- victim-side feedback;
- monitoring-side feedback;
- direct access to offensive and defensive tools.

![Tactical Cyber Operations Dashboard](Images_readme/tactical_cyber_operations_dashboard.png)

The dashboard also provides extended operational views for attack execution and feedback.



![Tactical Cyber Operations Dashboard - Operational Feedback](Images_readme/tactical_cyber_operations_dashboard_2.png)

The attack workflow is linked to detection and forensic expectations.

---


---

### 4.8 End-to-End Forensic Workflow View

The **End-to-End Forensic Workflow View** presents the forensic workflow as a complete process.

It helps the user understand the relation between incident execution, detection, acquisition, preservation, analysis, reporting, reconstruction, and comparison.

The workflow follows the sequence:

```text
incident execution
  -> detection
  -> trigger resolution
  -> acquisition
  -> preservation
  -> verification
  -> analysis
  -> reporting
  -> reconstruction
  -> comparison
```

This view connects operational activity with the case-centered DFIR process.

![End-to-End Forensic Workflow](Images_readme/End_to_End_Forensic_Workflow.png)

The next view executes evidence acquisition and preservation.

---

### 4.9 Forensic Acquisition Dashboard

The **Forensic Acquisition Dashboard** is used to acquire and preserve evidence from the deployed scenario.

It supports the creation of forensic cases from observed or triggered incidents.

The dashboard can manage:

- alert-triggered acquisition;
- manual acquisition when required;
- acquisition profile selection;
- case creation;
- memory acquisition;
- disk or snapshot preservation;
- network evidence import;
- alert preservation;
- OT/industrial evidence export;
- manifest generation;
- chain-of-custody registration;
- acquisition pipeline events.

![Forensic Acquisition Dashboard](Images_readme/forensic_acquisition_dashboard.png)

The dashboard also exposes detailed acquisition status and case-preservation information.

![Forensic Acquisition Dashboard - Case Preservation](Images_readme/forensic_acquisition_dashboard_2.png)

After acquisition, the preserved evidence can be inspected and analyzed.

---

### 4.10 Forensic Live Traffic Analyzer

The **Forensic Live Traffic Analyzer** supports network-oriented inspection during or after the incident workflow.

It helps the user interpret traffic observations and relate network activity to the preserved case.

It can support:

- traffic inspection;
- protocol-oriented analysis;
- network-event review;
- packet-level context;
- correlation with alerts and forensic case records.

![Forensic Live Traffic Analyzer](Images_readme/forensic_live_traffic_analyzer.png)

The next views expose the forensic report and analysis surfaces.

---

### 4.11 Digital Forensics Report and Analysis Dashboard

The **Digital Forensics Report and Analysis Dashboard** provides a case-centered view of preserved artifacts, integrity records, custody information, analysis outputs, and forensic summaries.

It allows the user to inspect:

- case identifier;
- preserved evidence inventory;
- evidence paths;
- manifest records;
- hash values;
- chain-of-custody entries;
- acquisition pipeline events;
- metadata records;
- analysis outputs;
- generated forensic summaries.

This dashboard separates primary evidence from derived analysis outputs, preserving the distinction between what was acquired and what was later produced by analysis.

![Digital Forensics Report](Images_readme/forensic_report.png)

Additional report views expose detailed case information and analysis outputs.



![Digital Forensics Report - Analysis View](Images_readme/forensic_report_2.png)



The preserved case can then be used for reconstruction and cross-run comparison.

---

### 4.12 FOC Reconstruction and Comparability View

The **FOC Reconstruction and Comparability View** organizes the preserved incident into a Forensic Observational Context and supports comparison across repeated executions.

The **Forensic Observational Context (FOC)** binds:

- incident window;
- trigger source;
- acquisition profile;
- preserved evidence layers;
- temporal anchors;
- integrity records;
- custody records;
- derived analysis outputs;
- uncertainty state;
- reconstruction criteria.

The view helps evaluate whether expected incident-to-evidence relations are:

- recovered;
- degraded;
- ambiguous;
- missing.

It also supports CPR-oriented interpretation, where Causal Path Recoverability summarizes how many expected causal relations are recoverable from preserved and analyzed evidence.

This view is important because it avoids treating preservation as equivalent to complete reconstruction. A case may be correctly preserved while still leaving some relations degraded, ambiguous, or unsupported.

![FOC Reconstruction Comparability](Images_readme/FOC_Reconstruction_Comparability.png)

The final service supports movement of selected artifacts to external laboratory machines.

---

### 4.13 Remote Lab Exchange Dashboard

The **Remote Lab Exchange Dashboard** supports controlled exchange of selected artifacts between FORGE-RFDT and external analysis environments.

It can be used to:

- export selected files to a remote laboratory machine;
- retrieve external analysis outputs;
- support external tool execution;
- preserve the distinction between platform-managed evidence and externally produced results.

This service is useful when specialized tools are executed outside the platform while the main experiment remains organized through FORGE-RFDT.

![Remote Lab Exchange Dashboard](Images_readme/LAB_EXCHANGE_DASHBOARD.png)

Repeated execution across many repetitions requires its own dedicated visibility layer, described next.

---

### 4.14 Campaign Repetitions Center

The **Campaign Repetitions Center** provides full execution transparency across repeated Level C, Level B, and Level A runs, accessed from the main dashboard's repetitions indicator.

For any single repetition, it renders a hierarchical process tree — from scenario destruction and redeployment down to per-instance tool installation, monitoring verification, attack execution, detection, case sealing, and multilayer analysis — each stage carrying real start/end timestamps, elapsed duration, and, where available, the historical baseline duration for that exact stage. Nested Level B and Level A executions appear inline within their parent Level C repetition, preserving the true execution hierarchy rather than flattening it.

Because a single repetition view is necessarily local, two complementary campaign-wide views are also available:

- a combined process tree spanning every repetition reached so far in the campaign;
- a summary table placing repetitions side by side, for comparing cases, tool-installation failures, detection outcomes, and blocking errors across the whole run.

Every failure surfaces with a concrete reason drawn from the underlying execution record; when no specific cause was captured, the view says so explicitly rather than remaining silent. This supports rapid, evidence-grounded diagnosis of where and why a given repetition diverged, without manual log inspection.

![Campaign Repetitions Center](Images_readme/Repetitions_Center_Rep_Level_c.png)

Completed campaigns can then be replayed as a fully navigable 3D reconstruction.

---

### 4.15 3D Campaign Reconstruction View

The **3D Campaign Reconstruction View** replays an already-completed campaign as a chronologically organized, navigable 3D scene using recorded execution events. This is **historical replay, not live physical-machine control**. The view follows instance deployment, tool preparation, incident execution, detection, evidence acquisition and sealing, analysis, causal reconstruction and campaign comparison.

The existing view includes:

- a hierarchical 3D navigation path from campaign to repetition, case and host;
- animated flow indicators between workflow stages;
- an Alert Center highlighting recorded attacks, detections, fault-injection results and metrics;
- per-repetition and campaign summaries for evidence coverage and reconstruction;
- play, pause, step, reset, seek and playback speed controls;
- a digital twin watcher reporting whether recorded divergences were detected and later resolved.

![3D Campaign Reconstruction View](Images_readme/3d_compaing.png)

![3D Campaign Reconstruction View - Two-Repetition Campaign Replay](Images_readme/3D_Campaign_Reconstruction_View.png)

**FORGE-RFDT extension:** when real industrial devices and their twins become connected, replay records should identify physical versus virtual event sources, preserve original timestamps and mark observational gaps. Replay must remain isolated from live control outputs.

---

### 4.16 Digital Twin View

The existing **Digital Twin View** continuously observes the deployed scenario and compares recorded expectations with reported current state. Its checks include configuration, service health, clock synchronization, network exposure, resources and available industrial-process indicators. The monitoring view is **read-only**: detecting a discrepancy does not authorize or trigger changes to physical equipment.

The view keeps source-linked divergence events, their severity, resolution status and observational history. The intended physical-infrastructure extension will map live controller and machine telemetry to 3D twins, while explicitly distinguishing **expected**, **commanded**, **measured** and **visualized** states.

---

### 4.17 Fault Injection Dashboard

The existing **Fault Injection Dashboard** is designed for controlled, reversible fault exercises on deployed laboratory nodes. It provides fault profiles, rollback behavior, experiment-linked and manual launches, control/treatment comparison and effectiveness indicators showing whether the disturbance occurred and was noticed by the digital twin.

Supported examples in the documented virtualized baseline include configuration drift, detection-service interruption, clock skew and resource exhaustion. **These laboratory fault mechanisms must not be assumed safe on real PLCs, robotic systems or production machinery.** Physical fault injection requires independent approval, device-specific safety limits and rollback verification.

---

### 4.18 Real and Hybrid Industrial Asset Integration (Integration Target)

The extended infrastructure model should register real PLCs, industrial robots, sensors, IoT gateways, HMIs and their related virtual nodes with stable asset identifiers. Each asset needs a physical/virtual designation, accessible interfaces, authorization information, clock source, operational constraints and supported evidence sources. A virtual PLC linked to a virtual arm must remain distinguishable from a physical PLC commanding a real actuator.

### 4.19 Industrial 3D Digital Twin and Signal Mapping (TwinAxisLab Integration Target)

The intended [TwinAxisLab](https://github.com/Younyat/TwinAxisLab) integration adds 3D asset preparation, joints and kinematic representations, functional components, assemblies, signal mapping, industrial connectivity, monitoring and historical replay. Signal bindings should capture protocol, endpoint, address, units, scaling, access mode, source quality and timestamp. A 3D animation is a representation and does not itself prove physical motion. Write or motion commands to real equipment must remain explicitly authorized and interlocked.

### 4.20 Forensic State Divergence and Evidence Correlation (FORGE-DT Integration Target)

The intended [FORGE-DT](https://github.com/Younyat/FORGE-DT) integration will associate reference-state models with real observations, generating attributable divergence events, uncertainty indicators and supporting forensic artifacts. Links between a PLC command, network traffic, sensor reading, machine response and twin visualization should be reconstructed only where evidence supports the relation. A discrepancy is not, by itself, proof of a cyberattack.

### 4.21 RF Acquisition and Spectrum Monitoring (Integration Target)

The **RF Evidence Engine** is intended to make radio observations first-class case artifacts rather than screenshots or untracked signal traces. Supported receiver backends should be registered explicitly (e.g., suitable USRP/SDR devices and protocol-specific sniffers); actual hardware, bandwidth and protocol support depend on the connected backend and validation status.

The target includes:

- **Raw IQ acquisition:** continuous, scheduled, triggered and ring-buffer-assisted captures, subject to receiver throughput and storage limits.
- **Spectrum observations:** frequency/time power representations, occupancy, burst onset and end times, noise-floor estimates and RF anomaly candidates.
- **Wireless observations:** eligible BLE, Zigbee/IEEE 802.15.4, Wi-Fi or other IoT packet captures only where an appropriate channel-aware receiver, decoder and lawful access are available.
- **Receiver provenance:** hardware/model, serial or pseudonymous identifier, antenna, gain, center frequency, sample rate, analog/digital filters, clock/time source, software version and calibration details when available.
- **Uncertainty and coverage:** gaps, dropped samples, retunes, overload, clock quality, channels not observed and demodulation errors should be reported, not hidden.
- **Case binding:** every RF segment or frame collection should reference the case, capture session, equipment and acquisition profile that produced it.

**Status:** architectural integration target; RF data must not be described as already preserved by the current case backend without implementation evidence.

### 4.22 RF Forensic Analysis and SpectraRFX Interoperability (Integration Target)

The planned [SpectraRFX](https://github.com/Younyat/SpectraRFX) interoperability layer should import **original IQ files** and associated receiver metadata for retrospective analysis. Intended analytical outputs include time-frequency spectrograms, detected transmissions, occupied bandwidth, channel activity, received-power trends and supported protocol-level frame interpretation. An RF event is not automatically an identified device, a decoded message or proof of malicious action.

Record the **tool version, parameters, processing time and source-file hashes** for each derived product. If RF fingerprinting or machine learning is applied, preserve the model/version and confidence and treat its identity decisions as hypotheses requiring independent supporting evidence.

### 4.23 RF Evidence Preservation and Chain of Custody (Integration Target)

RF evidence should follow the same distinction between **primary** and **derived** material used by FORGE. An appropriate interchange structure is raw IQ with [SigMF](https://sigmf.org/) metadata where supported, and PCAP/PCAPNG for decoded radio frames when the protocol and capture backend permit it.

The acquisition system should record UTC timestamps, timestamp origin/precision, start/stop times, sample count, frequency coverage, receiver configuration, acquisition gaps, cryptographic digests, artifact lineage, custody actions and any post-acquisition transformations. Re-analysis must not overwrite the original RF evidence.

### 4.24 Cross-Domain RF–IT/OT–IoT–DT Reconstruction (Integration Target)

The reconstruction engine should combine **RF bursts/frames**, **gateway and network events**, **PLC commands and process measurements**, **robot/sensor observations** and **digital-twin expected/observed states** into one case-centered evidence graph. It should distinguish:

- **Observed:** a captured RF burst, packet, PLC register reading or sensor measurement.
- **Derived:** a decoded frame, estimated transmission feature, inferred source or twin discrepancy.
- **Expected:** a scenario reference model or declared causal relation.
- **Unresolved:** incomplete captures, unsynchronized timestamps or competing explanations.

Timing overlap alone cannot establish that a specific radio transmitter caused a PLC change. Device attribution and causal edges require protocol, host, gateway or other independent corroboration. Existing FOC and CPR/WCPR interpretations should be extended only with documented RF relation types and scoring rules.

### 4.25 RF Replay and Case Review (Integration Target)

The proposed RF review surface combines synchronized event markers with spectrum/waterfall and IQ-derived observations, plus the existing 3D campaign replay and industrial digital-twin views. The intended replay is **offline and non-actuating**: no stored radio transmission, PLC command or robot action may be emitted to the physical environment as a consequence of scrubbing the timeline.



---

## 5. Evidence Model

FORGE-RFDT treats evidence handling as part of the experimental procedure. Each incident execution can materialize as a forensic case under the evidence store.

A case is identified by a stable case identifier and contains preserved artifacts, metadata, integrity records, custody records, pipeline events, and derived analysis outputs.

### 5.1 Case as Experimental Unit

In FORGE-RFDT, the forensic case is the unit used for preservation, analysis, reconstruction, and comparison.

A case records:

- what incident triggered acquisition;
- what profile was selected;
- what evidence was collected;
- what artifacts were hashed;
- what custody events were recorded;
- what analysis outputs were generated;
- what reconstruction criteria were applied;
- what comparison outputs were exported.

### 5.2 Evidence-Store Layout

A typical case follows a structured layout such as:

```text
evidence_store/
  CASE-YYYYMMDD-HHMMSS/
    manifest.json
    chain_of_custody.log
    metadata/
    alerts/
    network/
    memory/
    disk/
    industrial/
    pipeline/
    timelines/
    derived/
    reports/
```

The exact content depends on the acquisition profile and scenario type.

### 5.3 Primary and Derived Artifacts

FORGE-RFDT separates primary evidence from derived artifacts.

Primary evidence includes preserved artifacts such as:

- memory images;
- disk images or snapshots;
- PCAP files;
- raw alerts;
- OT exports;
- metadata;
- custody records;
- hash records.

Derived artifacts include outputs generated by analysis, parsing, enrichment, reconstruction, or reporting.

Examples include:

- parsed network summaries;
- memory-analysis outputs;
- disk-analysis outputs;
- normalized timelines;
- cross-layer findings;
- reconstruction reports;
- executive summaries;
- comparison exports.

The next section explains how acquisition profiles populate this evidence model.

---

### 5.4 Physical-Asset and Digital-Twin Evidence (Target Model)

A future physical/hybrid case should distinguish **original observations** (captured controller telemetry, sensor readings, network traces, field-system logs, timestamps and source metadata) from **derived records** (3D state visualizations, reconstructed motion, calculated state divergence and explanatory timelines). The case should identify the observation mechanism, sampling interval, clock synchronization, missing intervals, uncertainty and device/protocol details.

A target evidence package can add optional locations without changing the legacy case layout:

```text
physical_assets/      # real equipment inventory, controller/gateway observations
industrial_signals/   # timestamped signal values and quality metadata
digital_twin/         # versioned twin definition, state traces, mappings and divergence outputs
radio/                # optional future IQ and wireless capture evidence
```

These directories describe an integration schema and do not imply that the current acquisition backend already creates them.

### 5.5 Radio-Frequency Evidence (Target Model)

Each RF case should identify both the **unmodified source capture** and the **derived analytical results**. The proposed directory extension (illustrative; not a claim about existing on-disk implementation) is:

```text
evidence_store/
  CASE-YYYYMMDD-HHMMSS/
    rf/
      sessions/
        RFSESSION-ID/
          acquisition.json          # receiver, clock, antenna, band and coverage
          capture.sigmf-meta        # metadata when SigMF is used
          capture.sigmf-data        # original IQ samples (example)
          frames.pcapng             # when frame capture is supported
          capture_events.jsonl      # sample loss, retune, clock events
      derived/
        rf_events.jsonl
        spectrum_summary.json
        protocol_analysis.json
        rf_timeline.json
      reports/
        rf_case_summary.md
```

The existing case manifest and chain-of-custody files remain authoritative for artifact registration and integrity. RF source identifiers should be retained alongside clock uncertainty and observed coverage: absence of a detected transmission outside a measured channel or capture window is **not** evidence that transmission never occurred.

---

## 6. Acquisition and Preservation Model

FORGE-RFDT uses profile-conditioned acquisition. This means that the selected trigger and scenario context determine which acquisition procedure is executed.

### 6.1 Alert-to-Profile Resolution

The acquisition workflow follows this chain:

```text
Alert
  -> trigger identifier
  -> acquisition profile
  -> versioned procedure
  -> acquisition backend
  -> preserved case
```

The platform records:

- trigger time;
- trigger source;
- selected profile;
- selected procedure;
- parameters;
- acquisition start events;
- preservation events;
- outcomes;
- diagnostic context.

### 6.2 Evidence Sources

Depending on the active profile, FORGE-RFDT can preserve:

- network traffic;
- volatile memory;
- persistent host state;
- disk or snapshot artifacts;
- detector alerts;
- OT protocol exports;
- SCADA/HMI logs when exposed;
- metadata and timing records;
- acquisition pipeline events.

### 6.3 OT and Industrial Evidence

For IT/OT scenarios, the acquisition model includes industrial evidence beyond generic PCAP storage.

The platform can preserve structured OT exports such as:

- selected register ranges;
- selected coil ranges;
- setpoints;
- I/O values;
- protocol-specific observation metadata;
- address maps;
- sampling policy;
- UTC timestamps.

The evaluated industrial baseline is centered on Modbus/TCP-style observation, while the case-preservation structure is designed to remain protocol-agnostic at the evidence-store level.

### 6.4 Integrity and Custody

FORGE-RFDT records preservation and handling steps through:

- manifest entries;
- cryptographic hashes;
- chain-of-custody records;
- acquisition pipeline events;
- metadata files;
- verification outputs.

The platform can expose inconsistencies between declared actions and preserved records by keeping acquisition, preservation, and verification records inspectable.

The next section describes how controlled incident profiles connect the attack layer to detection and forensic expectations.

---

## 7. Attack and Detection Model

FORGE-RFDT uses controlled attack profiles to generate incident conditions under documented constraints.

Each attack profile can define:

- technique identity;
- target role;
- execution backend;
- safety constraints;
- expected detector;
- expected alert;
- expected forensic artifacts;
- rollback behavior;
- DFIR escalation flag;
- reconstruction expectations.

### 7.1 Controlled Incident Execution

The attack layer is designed for reproducible laboratory execution.

It supports:

- target selection;
- profile-constrained execution;
- role validation;
- backend script execution;
- output capture;
- rollback where required;
- linkage to expected evidence.

### 7.2 Detection Linkage

Detection profiles connect controlled attacks to observable evidence.

A detection workflow can preserve:

- detector-native alerts;
- normalized alert views;
- alert-to-trigger mappings;
- Wazuh events;
- Suricata outputs;
- OT-specific detection outputs;
- correlation metadata.

### 7.3 Safety and Scope

FORGE-RFDT attack profiles are intended for controlled laboratory scenarios deployed by the platform.

They are not intended for production systems, unauthorized networks, or uncontrolled environments.

The next section describes how preserved evidence is reconstructed and compared.

---

### 7.4 RF and Wireless Incident Profiles (Proposed)

RF-related scenarios may investigate interference, wireless communication outages, unexpected bursts, IoT message discrepancies and differences between a radio event and the corresponding gateway/PLC observation. Initial experiments should use passive acquisition and controlled transmitters only in permitted laboratory configurations. Emitting interference, retransmitting captured traffic or altering a physical machine is **not** part of a default acquisition workflow.

---

## 8. Reconstruction and Reproducibility

FORGE-RFDT separates preservation from reconstruction.

A case may be well preserved and still have degraded, ambiguous, or missing causal relations. The reconstruction layer makes this distinction explicit.

### 8.1 Reconstruction Objects

The reconstruction layer uses:

- incident window;
- preserved artifacts;
- analysis outputs;
- integrity records;
- custody records;
- trigger records;
- temporal metadata;
- expected relation models;
- uncertainty indicators.

### 8.2 Relation States

Expected relations are evaluated using explicit states:

```text
recovered
degraded
ambiguous
missing
```

These states help the platform express whether a relation is fully supported, partially supported, unresolved, or unsupported by the available evidence.

**Causal Path Recoverability (CPR)** and its weighted variant summarize these states into two comparable metrics:

```text
CPR  = fully recovered expected causal edges / total expected causal edges
WCPR = recovered edge weight / total expected edge weight
```

CPR treats every expected edge equally; WCPR uses the scenario-defined per-edge weights, so structurally more important relations count more. Only `recovered` edges contribute to either numerator — `degraded` and `ambiguous` edges contribute 0, same as `missing`. Temporal and degradation penalties affect reconstruction confidence separately; they do not alter CPR or WCPR themselves. WCPR is the metric compared across repeated Level C deployments (ΔWCPR) to assess reproducibility.

### 8.3 Reproducibility Levels

FORGE-RFDT supports repetition-oriented organization through:

```text
Level A -> re-analysis of the same sealed case
Level B -> repeated incident execution in the same deployed scenario
Level C -> redeployment-aware repetition and comparison
```

These levels allow the user to distinguish analytical repeatability, incident repetition, and redeployment-aware experimental comparison.

### 8.4 Scientific Memory and Comparison Families

FORGE-RFDT can organize repeated outputs into comparison families.

A comparison family can group cases according to:

- scenario profile;
- attack profile;
- acquisition profile;
- repetition level;
- evidence coverage;
- reconstruction criteria;
- exported scientific result profile.

This makes repeated executions easier to inspect and compare without mixing incompatible campaigns.

The next section gives a practical end-to-end usage sequence.

---

### 8.5 Radio-to-Physical Reconstruction Example (Illustrative)

An illustrative causal hypothesis is **IoT sensor wireless transmission → gateway receipt → PLC state transition → robot response → digital-twin observation**. The RF receiver might capture a burst, a protocol sniffer might decode a frame, and IT/OT logs might register the downstream events. The reconstruction engine must show which links are independently supported and where missing bands, channels, timestamps or payload visibility prevent confirmation.

---

## 9. End-to-End Usage Sequence

A complete FORGE-RFDT workflow can be executed as follows.

### Step 1: Deploy the Infrastructure

```bash
bash openstack-installer/openstack-installer.sh
```

### Step 2: Start the Platform

```bash
bash start_dashboard.sh
```

### Step 3: Create the Base IT Scenario

Use the **IT Scenario Editor** to create nodes, assign roles, connect the topology, and deploy the scenario.

### Step 4: Extend the Scenario with OT Components

Use the **Industrial Scenario Editor** to add PLC, SCADA, or HMI components when the experiment requires an IT/OT setting.

### Step 5: Install Tools

Use the **Instance Tools Manager** to install required tools on the attacker, victim, monitor, PLC, SCADA/HMI, or forensic host.

### Step 6: Access Operational Tools

Use the **Security Training and Tools Portal** to open installed tools and interact with the deployed environment.

### Step 7: Execute a Controlled Incident

Use the **Attack Lab** or **Tactical Cyber Operations Dashboard** to select a target and launch a controlled attack profile.

### Step 8: Observe Detection

Use the monitoring and detection surfaces to observe whether the expected alert or telemetry was produced.

### Step 9: Acquire and Preserve Evidence

Use the **Forensic Acquisition and Analysis Dashboard** to trigger or execute the acquisition profile and create a preserved case.

### Step 10: Review the Case

Use the **Digital Forensics Report and Analysis Dashboard** to inspect the manifest, custody records, artifact inventory, pipeline events, and analysis outputs.

### Step 11: Reconstruct the Incident

Use the **FOC Reconstruction Dashboard** to bind the incident window, preserved artifacts, derived findings, uncertainty state, and expected relations.

### Step 12: Compare Repeated Executions

Use the **Forensic Repetition Manager** and **Forensic Reconstruction Comparability View** to compare cases generated under documented repetition conditions.

### Step 13: Replay a Completed Campaign in 3D

Use the **3D Campaign Reconstruction View** to inspect recorded events, alerts and causal stages in chronological order, without sending commands to physical equipment.

### Step 14: Review Digital Twin Divergences and Controlled Fault Results

Use the **Digital Twin View** and **Fault Injection Dashboard** to inspect observed deviations, their evidentiary support and permitted laboratory fault outcomes. Physical connections and safety approval are separate integration requirements.

The next section lists the main strengths exposed by this workflow.

---

### Step 15: Acquire and Preserve RF Evidence (Planned)

Register an authorized SDR or wireless sniffer, set receiver configuration and synchronization, capture the relevant frequency/channel windows, seal original IQ/frame files and link hashes, metadata and custody records to the case. **This step requires implementation and a supported acquisition backend.**

### Step 16: Analyze RF Evidence and Correlate with the Industrial Twin (Planned)

Inspect preserved samples and decoded frames offline, derive spectrum/wireless events, compare with IoT/PLC/network records and twin-state divergence and assign supported or uncertain relations. Export a report that cites original source artifacts rather than relying on plots alone.

---

## 10. Baseline Capabilities and Planned Extensions

The inherited virtualized baseline provides:

- automated OpenStack-based infrastructure deployment;
- visual IT scenario construction;
- visual IT/OT scenario extension;
- role-aware node configuration;
- centralized tool installation;
- direct access to operational security tools;
- ATT&CK-aligned controlled attack profiles;
- integrated detection and alert handling;
- profile-conditioned forensic acquisition;
- volatility-aware preservation ordering;
- rolling PCAP and case-bound network import;
- memory, disk, network, alert, and OT evidence preservation;
- manifest-based integrity metadata;
- hash-linked custody recording;
- primary and derived artifact separation;
- post-acquisition forensic analysis;
- FOC-based reconstruction;
- causal relation state tracking;
- CPR-oriented reconstruction summaries;
- Level A, Level B, and Level C repetition organization;
- cross-run comparability views;
- research and training usability.
- chronological **3D campaign reconstruction** with navigable replay and campaign-level reports;
- live, read-only **Digital Twin View** with divergence history;
- controlled laboratory **Fault Injection Dashboard** with effectiveness and rollback reporting.

The FORGE-RFDT integration roadmap additionally includes physical-asset onboarding, hybrid topologies, TwinAxisLab-derived 3D/kinematic digital twins, FORGE-DT-style state-divergence investigation, and **SDR/IQ acquisition, spectrum monitoring, wireless protocol captures, RF evidence preservation, offline analysis, RF-aware causal reconstruction and synchronized RF/3D replay**. These items are **integration targets until implemented and verified in FORGE-RFDT**.

The next section maps the documented baseline capabilities to repository paths.

---

## 11. Repository Structure

Important repository paths include:

```text
openstack-installer/
```

OpenStack deployment scripts and virtual environment setup.

```text
app_core/
```

Backend application logic, infrastructure services, attack catalog, forensic services, and orchestration components.

```text
app_core/infrastructure/attack/
```

Controlled attack profile catalog and execution support.

```text
PLC/
```

PLC-related resources, including prepared control programs.

```text
PLC/plc_programs/
```

Industrial control logic examples.

```text
Images_readme/
```

Images used by this README.

```text
evidence_store/
```

Case-level evidence store generated during forensic acquisition workflows.



The next section defines the intended scope of the platform.

---

### RF Data Volume and Retention

Uncompressed IQ can grow rapidly: **2 channels × 20 MS/s × 4 bytes per complex sample = 160 MB/s (decimal), or 576 GB/hour**. This is an illustrative planning figure, not an observed FORGE-RFDT result. Rates depend on sample format, channels, duty cycle and compression. Configure disk quotas, retention, ring buffers and artifact export before enabling sustained reception. Raw IQ files and IQ-derived exports should generally remain outside Git.

---

## 12. Scope

FORGE-RFDT is designed for controlled experimentation, research, training, and reproducibility-oriented DFIR workflows across virtual IT/OT environments and, subject to integration and safety validation, real and hybrid industrial infrastructures.

It is intended for:

- local laboratory environments;
- controlled cybersecurity experimentation;
- virtualized, physical, and hybrid IT/OT scenarios, subject to the availability of connectors and appropriate authorizations;
- educational exercises;
- forensic acquisition and preservation studies;
- post-incident reconstruction experiments;
- cross-run comparison of documented executions.

It is not intended for:

- unauthorized testing;
- uncontrolled production networks;
- safety-critical live industrial processes;
- certification of real-world forensic completeness;
- replacement of professional incident-response procedures in operational environments.

Physical process fidelity and observability depend on whether the case uses simulated equipment, instrumented real equipment, or both. The platform prioritizes controlled repetition, evidence traceability, auditable preservation, reconstruction support, and comparative analysis.

The next section describes safe repository hygiene before sharing or publishing deployments.

---
## 13. Repository Content and Artifact Management

The FORGE-RFDT repository retains the source code, scenario definitions, deployment configuration, attack and detection profiles, acquisition procedures, analysis components, documentation, and lightweight metadata required to understand and re-execute the latest documented experimental scenario.

## Sensitive and Environment-Specific Files

Files containing credentials, private keys, passwords, access tokens, local addresses, or deployment-specific configuration are excluded from the public repository.

Examples:

```text
admin-openrc.sh
*.pem
*.key
*.env
```

Sanitized templates are provided when configuration examples are required:

```text
admin-openrc.example.sh
.env.example
```

## Generated Development Files

Runtime logs, Python caches, virtual environments, dependency directories, editor settings, temporary files, and build outputs are treated as locally generated content and are excluded from version control.

Examples:

```text
__pycache__/
*.pyc
.venv/
venv/
node_modules/
dist/
.validation-build/
.idea/
.vscode/
app.log
app.log.*
wget-log
```

## Large Experimental and Forensic Artifacts

A FORGE-RFDT execution can generate large forensic artifacts, including memory images, disk images or snapshots, network captures, industrial evidence exports, case evidence collections, analysis outputs, and execution-event records.

Typical storage consumption for a completed case, per artifact type:

| Component | Average size | Content |
|---|---|---|
| `disk_raw` | 70.00 GB | Full raw disk images of the preserved nodes |
| `network_pcap` | 12.38 GB | Continuous rolling network captures spanning the attack window — the only component that varies with capture duration |
| `memory_lime` | 8.00 GB | Volatile memory dump(s) in LiME format |
| Metadata, logs, and manifests | < 0.01 GB combined | Chain of custody, hashes, IR inputs, acquisition profile, and related records |
| **Total per case** | **≈ 90.4 GB** | |

These generated artifacts are retained outside normal Git version control when their size is not compatible with public repository distribution. GitHub blocks individual files larger than 100 MiB in regular Git repositories.

Common excluded artifacts and locations include:

```text
*.pcap
*.pcapng
*.raw
*.mem
*.lime
*.img
*.dd
*.qcow2
*.jsonl
evidence_store/
```

The exclusion of these files applies to the generated outputs of an execution, not to the experimental procedure used to produce them.

## Experiment Re-execution and Artifact Regeneration

The latest documented scenario definition remains registered in the repository together with its deployment configuration, automation scripts, incident profile, detection expectations, acquisition profile, analysis procedures, and lightweight execution metadata.

These retained elements provide the basis for redeploying the scenario, repeating the experiment, and regenerating the corresponding forensic artifacts and derived reports.

Lightweight reports from the latest documented executions are included when their size and content are compatible with normal repository distribution. Large evidence files and other execution outputs are not included directly in the public repository.

The repository therefore preserves:

```text
scenario and topology definitions
deployment and teardown procedures
attack and detection profiles
forensic acquisition profiles
analysis and reconstruction procedures
execution parameters
lightweight manifests and metadata
latest lightweight reports
artifact generation procedures
```

The repository does not necessarily preserve:

```text
complete memory images
complete disk images or snapshots
large PCAP collections
full evidence-store contents
large execution-event logs
other generated files exceeding repository limits
```

Regenerated artifacts are not assumed to be byte-identical to those produced in an earlier execution. Their content can depend on infrastructure state, timing, software versions, runtime conditions, and the evidence observed during each repetition.

Small reference artifacts can be included when their purpose is documented, they contain no sensitive information, and their size is compatible with normal repository distribution.

The next section describes where the full evidentiary package for the reported campaign can be obtained.

---


## 14. Historical FORGE-VI Verification Dataset

The historical **FORGE-VI virtualized baseline** verification dataset is distributed separately through Zenodo. It must not be interpreted as validation of FORGE-RFDT physical-device integration, digital-twin interoperability, **RF acquisition, radio attribution or RF-aware causal reconstruction**.

The historical virtualized verification dataset contains:

- one complete sealed forensic case from the reported reconstruction campaign;
- campaign-level records for all ten reported executions;
- preserved evidence, case metadata, manifests, integrity records, and custody information;
- analysis and reconstruction outputs, together with cross-run summaries.

These materials support independent inspection of the preserved evidence and verification of the reported campaign results without requiring infrastructure redeployment or re-execution of the experiment.

| | |
|---|---|
| **Repository** | [Zenodo — FORGE-VI Verification Dataset](https://doi.org/10.5281/zenodo.22706525) |
| **DOI** | [10.5281/zenodo.22706525](https://doi.org/10.5281/zenodo.22706525) |
| **Dataset version** | 1.0.0 |
| **Campaign ID** | `CMP-20260903-083211-125A` |
| **Access** | Public, open access |

The next section acknowledges the funding and institutional support behind the project.

---

## 15. Related Projects and Scientific Boundaries

- [FORGE-RI](https://github.com/Younyat/FORGE-RI): real/virtual/hybrid IT/OT forensic infrastructure direction.
- [FORGE-DT](https://github.com/Younyat/FORGE-DT): digital-twin state observation and divergence investigation.
- [TwinAxisLab](https://github.com/Younyat/TwinAxisLab): 3D industrial asset models, kinematics, signal bindings and digital-twin engineering.
- [SpectraRFX](https://github.com/Younyat/SpectraRFX): RF spectrum monitoring and radio analysis project proposed as the RF interoperability source.
- [SigMF](https://sigmf.org/): metadata convention for recorded signal samples.

The term **complete reconstruction** describes the intended *multi-domain scope*, not guaranteed full observability. Claims of acquisition completeness, correct emitter identification, precise timing and causal reconstruction should be reported only for evaluated scenarios with known receiver coverage, ground truth and measurable error.

---

## 16. Acknowledgments

This repository is part of the project **“CiberIA: Investigación e Innovación para la Integración de Ciberseguridad e Inteligencia Artificial”** (Proyecto C079/23), financed by the European Union NextGeneration-EU, the Recovery Plan, Transformation and Resilience, through INCIBE.

This work is also supported by the **Programa Global de Innovación en Seguridad** for the promotion of Cátedras de Ciberseguridad en España, funded by the European Union NextGeneration-EU Funds through the Instituto Nacional de Ciberseguridad (INCIBE).

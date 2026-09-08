# Cloud-Based Linux Security Monitoring Lab

## Wazuh Cloud + Kali Linux + VirtualBox

A hands-on cybersecurity lab demonstrating cloud-based Linux endpoint monitoring, vulnerability detection, SSH authentication monitoring, and File Integrity Monitoring using Wazuh Cloud.

---

## Project Overview

This project demonstrates the deployment and use of a Wazuh security monitoring environment with a Kali Linux virtual machine.

A Wazuh Agent was installed on the Kali Linux endpoint and connected to Wazuh Cloud. Security telemetry was then collected and investigated through the Wazuh Cloud dashboard.

The lab focused on four primary security monitoring capabilities:

* Wazuh Agent deployment and cloud connectivity
* Vulnerability detection
* SSH authentication monitoring
* File Integrity Monitoring (FIM)

A custom Wazuh detection rule was also attempted as part of the detection-engineering process. The custom rule did not produce the expected alert, so the issue was documented as a troubleshooting exercise rather than being presented as a successful detection.

---

## Objectives

The main objectives of this project were to:

1. Deploy a Wazuh Agent on a Linux endpoint.
2. Connect the endpoint to Wazuh Cloud.
3. Verify that security telemetry was being received.
4. Identify vulnerabilities present on the endpoint.
5. Monitor SSH authentication activity.
6. Generate controlled authentication-failure events.
7. Configure and test File Integrity Monitoring.
8. Investigate Wazuh security alerts.
9. Attempt to create a custom detection rule.
10. Document the investigation and troubleshooting process.

---

## Lab Environment

| Component                | Details                                  |
| ------------------------ | ---------------------------------------- |
| SIEM / Security Platform | Wazuh Cloud                              |
| Endpoint                 | Kali Linux                               |
| Kali Version             | Kali GNU/Linux 2024.3                    |
| Wazuh Agent              | 4.14.7                                   |
| Virtualization           | VirtualBox                               |
| Agent Name               | `kali-lab`                               |
| Agent ID                 | `001`                                    |
| Monitoring               | Vulnerabilities, SSH authentication, FIM |

---

## Architecture

```text
                 ┌─────────────────────────────┐
                 │       Kali Linux VM         │
                 │                             │
                 │  SSH Authentication Logs    │
                 │  File Changes               │
                 │  System Activity            │
                 │  Vulnerable Packages        │
                 └──────────────┬──────────────┘
                                │
                                │ Wazuh Agent
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │        Wazuh Cloud          │
                 │                             │
                 │  Alert Detection            │
                 │  Vulnerability Detection    │
                 │  FIM                       │
                 │  Event Investigation        │
                 └──────────────┬──────────────┘
                                │
                                ▼
                 ┌─────────────────────────────┐
                 │       SOC Investigation     │
                 │                             │
                 │  Analyze Events             │
                 │  Identify Threats           │
                 │  Determine Severity         │
                 │  Recommend Response         │
                 └─────────────────────────────┘
```

---

# Security Monitoring Capabilities

## 1. Wazuh Agent Deployment

A Wazuh Agent version 4.14.7 was installed on the Kali Linux virtual machine and successfully connected to Wazuh Cloud.

The endpoint appeared in Wazuh Cloud as:

* Agent name: `kali-lab`
* Agent ID: `001`
* Operating system: Kali GNU/Linux 2024.3

This confirmed that the endpoint was successfully enrolled and communicating with the Wazuh environment.

### Evidence

See:

`screenshots/01-agent-connected.png`

---

## 2. Vulnerability Detection

Wazuh vulnerability detection was used to identify security vulnerabilities affecting packages installed on the Linux endpoint.

One of the findings identified during the lab was:

| Field    | Finding        |
| -------- | -------------- |
| CVE      | CVE-2025-69872 |
| Package  | `diskcache`    |
| Severity | Critical       |
| CVSS     | 9.8            |
| Endpoint | `kali-lab`     |

The finding demonstrates how endpoint vulnerability information can be surfaced through a centralized security monitoring platform.

The lab did not attempt to exploit the vulnerability. The finding was treated as a security-monitoring and vulnerability-management observation.

### Evidence

See:

`screenshots/03-vulnerability-detection.png`

---

## 3. SSH Authentication Monitoring

SSH authentication monitoring was tested by generating controlled failed login attempts against the local SSH service.

Example test:

```bash
ssh fakeuser@localhost
```

An intentionally nonexistent username was used to generate an authentication failure.

Wazuh detected the activity using its built-in SSH detection rule:

```text
Rule ID: 5710
Rule Level: 5
```

Detection:

```text
sshd: Attempt to login using a non-existent user
```

The event included information such as:

```text
User: fakeuser
Source: ::1
```

The alert was associated with authentication-failure activity and MITRE ATT&CK techniques including:

```text
T1110.001 - Password Guessing
T1021.004 - SSH
```

The test was performed locally in a controlled lab environment and was not an attempt to compromise an external system.

### Evidence

See:

`screenshots/02-ssh-authentication.png`

---

## 4. File Integrity Monitoring

File Integrity Monitoring was configured to monitor important Linux directories.

The monitored locations included:

```text
/etc
/usr/bin
/usr/sbin
/bin
/sbin
/boot
```

Real-time monitoring was enabled for `/etc` during the lab test.

A controlled test file was created:

```text
/etc/wazuh-fim-test.txt
```

The file was modified to generate a FIM event.

Wazuh detected the change with:

```text
Rule ID: 550
Rule Level: 7
```

Detection:

```text
Integrity checksum changed.
```

The event identified:

```text
syscheck.event: modified
```

and the affected path:

```text
/etc/wazuh-fim-test.txt
```

The test file was created solely for the lab and was subsequently removed.

### Evidence

See:

`screenshots/04-file-integrity.png`

---

# Detection Summary

| Capability                    | Result                                  |
| ----------------------------- | --------------------------------------- |
| Wazuh Agent Connection        | Successful                              |
| Vulnerability Detection       | Successful                              |
| SSH Authentication Monitoring | Successful                              |
| File Integrity Monitoring     | Successful                              |
| Custom Detection Rule         | Attempted / Requires Further Validation |

---

# Custom Detection Rule Attempt

As part of the detection-engineering portion of the project, a custom Wazuh rule was attempted.

The objective was to create a higher-severity custom detection based on the built-in SSH rule:

```text
Rule ID: 5710
```

The attempted custom rule used:

```text
Rule ID: 100001
Level: 10
```

The custom rule was deployed through the Wazuh Cloud rule-management interface and the Wazuh configuration was reloaded.

However, subsequent controlled SSH tests continued to trigger the built-in rule:

```text
5710
```

rather than the expected custom rule:

```text
100001
```

The custom rule was therefore not considered successful.

This was documented as a troubleshooting exercise to demonstrate the importance of validating rule deployment, syntax, rule dependencies, and cloud-side processing before considering a detection-engineering task complete.

---

# Investigation Process

The general investigation workflow used during this lab was:

```text
Generate Controlled Event
        ↓
Wazuh Agent Collects Telemetry
        ↓
Wazuh Cloud Processes Event
        ↓
Detection Rule Matches Event
        ↓
Alert Appears in Wazuh
        ↓
Investigate Event Fields
        ↓
Identify Source / User / Event
        ↓
Determine Security Relevance
        ↓
Recommend Response
```

---

# Security Analysis

The lab demonstrated several important SOC monitoring concepts.

### Authentication Monitoring

Repeated failed SSH authentication attempts can indicate:

* Password guessing
* Brute-force activity
* Unauthorized access attempts
* Misconfigured services
* Automated scanning

In a production environment, repeated authentication failures would warrant investigation of the source, frequency, targeted accounts, and whether any successful authentication occurred afterward.

### Vulnerability Monitoring

Critical vulnerabilities should be prioritized based on:

* Severity
* CVSS score
* Exposure
* Affected software
* Availability of patches
* Business importance of the affected system

### File Integrity Monitoring

Unexpected modifications to sensitive directories such as `/etc` can indicate:

* Configuration changes
* Malware activity
* Persistence mechanisms
* Unauthorized administrative activity
* System compromise

FIM alerts therefore require contextual investigation rather than automatically being treated as malicious.

---

# Challenges and Troubleshooting

Several practical issues were encountered during the project.

## System Log Collection

The expected:

```text
/var/log/auth.log
```

file was not available in the environment.

Authentication events were instead handled through the Linux systemd journal.

The Wazuh Agent log confirmed journal monitoring.

This demonstrated the importance of understanding the logging architecture of the operating system rather than assuming a specific log-file location.

---

## File Integrity Monitoring Frequency

The initial FIM configuration used scheduled monitoring.

For the controlled `/etc` test, real-time monitoring was enabled to allow the modification event to be detected quickly.

This highlighted the difference between scheduled integrity scans and real-time monitoring.

---

## Custom Rule Troubleshooting

The custom detection rule did not generate the expected alert.

The investigation confirmed that the original built-in SSH rule continued to trigger successfully.

Further validation would be required to determine whether the issue was related to rule deployment, rule syntax, rule loading, rule hierarchy, or cloud-side processing.

Rather than claiming that the custom detection worked, the project documents the result accurately as an unresolved troubleshooting exercise.

---

# Security Considerations

All security events in this project were generated within a controlled virtual-machine environment.

The SSH authentication tests were performed against the local Kali Linux system.

No external systems were targeted.

The vulnerability identified by Wazuh was observed for monitoring purposes and was not intentionally exploited.

Sensitive information such as passwords, API keys, authentication tokens, private keys, and other credentials should never be committed to this repository.

Screenshots should be reviewed and redacted before publication.

---

# Key Skills Demonstrated

This project demonstrates practical experience with:

* Wazuh
* SIEM concepts
* Linux security monitoring
* Endpoint monitoring
* Vulnerability management
* SSH authentication monitoring
* File Integrity Monitoring
* Security alert investigation
* Log analysis
* MITRE ATT&CK mapping
* Detection engineering
* Security troubleshooting
* Technical documentation

---

# Tools and Technologies

* Wazuh Cloud
* Wazuh Agent 4.14.7
* Kali Linux 2024.3
* VirtualBox
* SSH
* Linux systemd journal
* File Integrity Monitoring
* MITRE ATT&CK

---

# Project Evidence

Screenshots documenting the lab are available in the [`screenshots`](./screenshots/) directory.

### Agent Connection

![Wazuh Agent Connected](./screenshots/01-agent-connected.png)

### SSH Authentication Monitoring

![SSH Authentication Alert](./screenshots/02-ssh-authentication.png)

### Vulnerability Detection

![Vulnerability Detection](./screenshots/03-vulnerability-detection.png)

### File Integrity Monitoring

![File Integrity Monitoring](./screenshots/04-file-integrity.png)

---

# Project Report

The complete project report is available here:

[Cloud-Based Linux Security Monitoring Lab](./report/Cloud-Based-Linux-Security-Monitoring-Lab.pdf)

---

# Future Improvements

Potential improvements to the lab include:

* Adding a Windows endpoint
* Adding multiple Linux endpoints
* Creating and validating additional custom detection rules
* Building brute-force detection scenarios
* Monitoring privilege escalation activity
* Adding automated response actions
* Creating custom dashboards
* Integrating additional security telemetry
* Performing vulnerability remediation and rescanning
* Expanding MITRE ATT&CK coverage

---

# Conclusion

This project successfully demonstrated a cloud-based Linux security monitoring environment using Wazuh Cloud and a Kali Linux endpoint.

The lab successfully implemented and investigated:

* Wazuh Agent connectivity
* Vulnerability detection
* SSH authentication monitoring
* File Integrity Monitoring

A custom detection rule was also attempted and documented as an unresolved troubleshooting exercise.

The project provided practical experience with endpoint telemetry, security alert investigation, vulnerability monitoring, integrity monitoring, and basic detection engineering within a controlled SOC-style laboratory environment.

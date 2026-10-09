# SOC Lab

A hands-on security operations lab using Wazuh, Sysmon, Ubuntu Server,
and Windows Server to investigate security events, validate detections,
and identify telemetry gaps.

This repository documents my learning through controlled, authorized
tests. It includes scenario plans, investigation reports, observed
results, and reusable documentation templates.

## Start Here

Three investigations that best demonstrate the work:

- **[Encoded PowerShell detection](investigations/INV-003.md):**
  Followed a harmless encoded command from Sysmon process telemetry
  to a Wazuh alert mapped to MITRE ATT&CK T1059.001.
- **[Scheduled-task detection gap and mitigation](investigations/INV-004.md):**
  Identified missing Sysmon telemetry, enabled Windows Security auditing,
  and confirmed detection through Event 4698 and an existing Wazuh rule.
- **[Sysmon collection troubleshooting](investigations/INV-002.md):**
  Diagnosed an invalid event-channel subscription caused by an extra
  character in the configuration, then verified collection end to end.

## Lab Environment

| Component | Configuration / purpose |
| --- | --- |
| Virtualization | VirtualBox |
| Monitoring server | Ubuntu Server 24.04.5 VM, `wazuh-server` |
| Windows endpoint | Windows Server 2022 VM, `win-client` |
| Security monitoring | Wazuh manager and Windows agent |
| Endpoint telemetry | Sysmon v15.22 with the Neo23x0 configuration |
| VM communication | Host-only network, `192.168.56.0/24` |
| Investigation sources | Wazuh Discover, Windows Event Viewer, PowerShell, and Linux journal logs |

### Telemetry Flow

The Windows endpoint sends configured Sysmon and Windows event channels
through the Wazuh agent to the monitoring server. Ubuntu SSH activity
is monitored locally on `wazuh-server`.

Investigations compare endpoint evidence with Wazuh results to distinguish
between an activity not being logged, telemetry not being collected,
and a detection not matching.

The host-only network describes communication between the VMs;
it is not a claim that the entire lab is air-gapped.

## Documented Results

| Test | Observed result | Evidence |
| --- | --- | --- |
| SSH login attempts using a nonexistent user | Wazuh rule **5710** produced nine alerts after the manager was restarted and the test repeated. The initial collection failure's root cause remains unproven. | [INV-001](investigations/INV-001.md) |
| Sysmon event collection | An extra character in the configured channel path caused subscription failure. Correcting the path and restarting the agent restored collection. | [INV-002](investigations/INV-002.md) |
| Encoded PowerShell command | Sysmon Event **1** reached Wazuh and triggered rule **92057**, mapped to **T1059.001**. | [INV-003](investigations/INV-003.md) |
| Scheduled-task creation | No matching `schtasks.exe` process event was found in the tested Sysmon configuration. Enabling Windows Security auditing produced Event **4698**, covered by Wazuh rule **60228**. | [INV-004](investigations/INV-004.md) |
| Application-log clearing | Windows Event **104** in the **System** channel triggered Wazuh rule **63104**, mapped to parent technique **T1070**. | [INV-005](investigations/INV-005.md) |

These are results from specific authorized lab runs, not guarantees of
coverage across all implementations of an ATT&CK technique.

## Key Findings

### A working pipeline does not guarantee complete coverage

Successful Sysmon collection and PowerShell detection did not mean
scheduled-task creation was visible. Each scenario needed its own
endpoint and SIEM checks.

### Missing telemetry cannot be fixed by an alert rule alone

For scheduled tasks, the useful remediation was enabling another
telemetry source: Windows Security auditing.

Once Event 4698 was collected, an existing Wazuh rule already covered it.
An initially added custom rule duplicated that coverage and was removed.

The original Sysmon visibility gap remains; the mitigation uses a
different data source rather than changing the Sysmon configuration.

### Configuration should be verified, not only visually inspected

A channel path looked correct and synchronized to the agent, but an
extra character prevented subscription. Checking the exact string
length helped identify the problem.

### Observed evidence takes priority over predictions

The log-clearing scenario predicted T1070.001, but the observed Wazuh
alert mapped to T1070. The investigation records the actual mapping.

Similarly, Event 104 was observed in the System channel, not in the
Application log that was cleared.

## Investigation Workflow

1. Define the scenario, prerequisites, expected telemetry, and cleanup.
2. Run the test on an authorized lab system.
3. Check endpoint evidence independently of the SIEM.
4. Search Wazuh within the relevant time window.
5. Record the observed event, rule, and ATT&CK mapping where available.
6. Investigate missing telemetry or alerts.
7. Retest after remediation and document remaining limitations.

## Repository Structure

```text
soc-lab/
├── README.md
├── investigations/
│   ├── INV-001.md
│   ├── INV-002.md
│   ├── INV-003.md
│   ├── INV-004.md
│   └── INV-005.md
├── scenarios/
│   ├── SC-002.md
│   ├── SC-003.md
│   ├── SC-004.md
│   └── SC-005.md
├── runs/
│   └── results.csv
└── templates/
    ├── detection-card.md
    ├── investigation-report.md
    ├── results-table-header.csv
    ├── run-record.md
    └── scenario-card.md
```

- **[Investigations](investigations/):** Evidence, reasoning, verdicts,
  troubleshooting, and lessons learned.
- **[Scenarios](scenarios/):** Test objectives, expected telemetry,
  prerequisites, and cleanup plans.
- **[Results log](runs/results.csv):** Recorded scenario outcomes,
  gaps, and remediation actions.
- **[Templates](templates/):** Reusable formats for consistent documentation.

## Current Status

### Completed and documented

- Ubuntu monitoring VM and Windows endpoint setup
- Wazuh installation and initial hardening
- Windows agent connectivity
- Sysmon installation and collection troubleshooting
- SSH, encoded PowerShell, scheduled-task, and log-clearing investigations
- Scheduled-task visibility mitigation and successful retest

### Planned / not yet evidenced in this repository

- Registry Run-key scenario:
  [SC-005](scenarios/SC-005.md) is a plan, not a completed investigation.
- Additional scenario testing and detection validation
- Custom detections where testing demonstrates a genuine need

## Scope and Limitations

- This is a learning and portfolio lab, not a production SOC deployment.
- Detection results depend on the tested configuration and collected data.
- An alert matching authorized test activity does not establish malicious intent.
- The suspected SSH collection stall after a VM freeze was not conclusively proven.
- The scheduled-task mitigation does not resolve the underlying Sysmon gap.
- The repository documents individual tests; it is not a complete automated
  deployment or reproduction package.

## Safety

Run scenarios only on systems you own or are explicitly authorized to test.

Some scenarios alter system state or remove logs. In particular, clearing
the Application log deletes its existing records and is not reversible.
Use disposable lab systems and appropriate snapshots or backups.

Review evidence before publishing it, and remove credentials, tokens,
personal information, or sensitive infrastructure details.

## Related Projects

- **[TriageAI](https://github.com/jatintrace/triageai):**
  Offline-first event triage with deterministic rules and human-reviewed
  mock-AI explanations.
- **[SecureGuard](https://github.com/jatintrace/secureguard):**
  Local static-analysis tooling for Python and PHP.
- **[ProofSentinel](https://github.com/jatintrace/proofsentinel):**
  An early-development security testing and evidence harness.

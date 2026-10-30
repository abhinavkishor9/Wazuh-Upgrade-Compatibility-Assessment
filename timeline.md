
# Timeline - Wazuh Upgrade Compatibility Assessment

## Assessment Date: 30 October 2026

This timeline records the sequence of activities performed during the Wazuh upgrade compatibility assessment. Only timestamps available in the collected evidence are included. Activities without a captured execution time are recorded in sequence without assigning an estimated timestamp.

| Sequence | Activity | Observation / Result |
|---|---|---|
| 1 | Identified the Linux environment | Confirmed Rocky Linux 9.8 running under VMware. |
| 2 | Identified the package management system | Initial `dpkg` command failed because the system uses RPM. |
| 3 | Collected central component versions | Wazuh Manager, Indexer, and Dashboard identified as 4.13.1. |
| 4 | Identified Filebeat version | Filebeat 7.10.2 was installed. |
| 5 | Checked service status | Wazuh Manager, Indexer, Dashboard, and Filebeat were active and running. |
| 6 | Inspected network architecture | Identified VM IP `192.168.203.129`, host IP `192.168.203.1`, and relevant service listeners. |
| 7 | Checked Windows Agent version | Registry metadata identified Wazuh Agent 4.12.0; WazuhSvc was running. |
| 8 | Inspected Windows Agent configuration | Located `ossec.conf` and identified log collection, rootcheck, SCA, syscheck, and active-response sections. |
| 9 | Inspected Linux configuration dependencies | Identified Manager configuration, local rules, local decoder, and Filebeat configuration. |
| 10 | Inspected Indexer plugins | Collected installed OpenSearch plugin information; a complete plugin compatibility review remained pending. |
| 11 | Reviewed previous backup evidence | Located Lab 121 backup and staged recovery copy of the Windows Agent configuration. |
| 12 | Reviewed backup integrity | Confirmed the recorded SHA-256 hash for the backup; previous Lab 121 evidence established a matching staged recovery copy. |
| 13 | Checked central backup evidence | Found a local rules backup file; full central component backups and Indexer snapshots remained unverified. |
| 14 | Attempted API connectivity | Initial Windows TCP connection to port 55000 failed despite successful ping. |
| 15 | Addressed API network access | Corrected API listener/firewall access and restored TCP connectivity from Windows. |
| 16 | Tested API endpoints | An Agents API request returned two affected agent records with no failed items. |
| 17 | Investigated API authentication | Located the installation credentials archive and extracted the nested password file. |
| 18 | Attempted API authentication | Authentication returned `Invalid credentials`; a valid bearer token was not obtained. |
| 19 | Recorded compatibility baseline | Populated the assessment with Agent 4.12.0, central components 4.13.1, and proposed target 4.14.9. |
| 20 | Documented assessment outcome | Identified remaining compatibility, configuration, backup, and authentication checks. |

## Recorded Timestamps

| Date and Time | Evidence |
|---|---|
| 15 October 2026, 06:34:50 | Last-modified timestamp of the Windows Agent `ossec.conf` file. |
| 21 October 2026, 07:54 | Recorded last-modified date and time of the Linux Manager `ossec.conf` file. |
| 23 October 2026, 06:56 | Recorded timestamp of `local_rules.xml.bak`. |
| 29 October 2026, 07:12:43 | Lab 121 backup directory timestamp: `20261029-071243`. |
| 30 October 2026, 06:07:47 | Wazuh Manager service start time shown in the captured service-status output. |
| 30 October 2026, 06:55:21 | Initial PowerShell assessment object creation. |
| 30 October 2026, 06:57:09 | Populated assessment object with component versions and proposed target version. |

## Final Assessment Status

**Decision: Further investigation required.**

The version baseline and several configuration and backup dependencies were established. API TCP connectivity was restored, but authentication remained unsuccessful. Full central backup readiness, Indexer snapshot validation, and target-specific configuration compatibility were not established.

No Wazuh upgrade was performed.

## Scope Limitations

- The timeline does not assign execution times to activities where timestamps were not captured.
- The Windows Agent version was identified through installed-application metadata.
- The central component versions were identified through RPM package metadata.
- Backup validation for the Windows Agent configuration was performed in Lab 121.
- Indexer snapshot readiness and complete central component recovery were not demonstrated.
- The proposed target version was used for assessment planning and was not an approved upgrade target.

# Investigation Notes 
## 1. Environment and Version Baseline

### Windows Endpoint

The Windows endpoint was running Windows 11 Pro with the Wazuh Agent installed.

The installed-application registry metadata identified:

- Product: Wazuh Agent
- Version: 4.12.0

The Windows service inventory showed:

- Service name: `WazuhSvc`
- Display name: `Wazuh`
- Status: Running

### Linux Wazuh Server

The server environment was identified as:

- Operating system: Rocky Linux 9.8 (Blue Onyx)
- Kernel: 5.14.0-687.31.1.el9_8.x86_64
- Virtualization: VMware
- Architecture: x86-64
- Network interface: `ens160`
- IP address: `192.168.203.129/24`

RPM package metadata established the following versions:

| Package | Installed Version |
|---|---|
| wazuh-manager | 4.13.1-1 |
| wazuh-indexer | 4.13.1-1 |
| wazuh-dashboard | 4.13.1-1 |
| filebeat | 7.10.2-2 |

The three central Wazuh components shared the same version, 4.13.1.

The Windows Agent was on version 4.12.0, one minor release behind the central components.

## 2. Service and Network Observations

The service inventory showed the following services in an active and running state:

- `wazuh-manager.service`
- `wazuh-indexer.service`
- `wazuh-dashboard.service`
- `filebeat.service`

The listener inspection identified:

| Port | Observed Service / Purpose | Listener |
|---|---|---|
| 443 | Wazuh Dashboard | 0.0.0.0 |
| 55000 | Wazuh API | 0.0.0.0 and IPv6 |
| 9200 | Wazuh Indexer API | Loopback |
| 1514 | Wazuh Agent communication | 0.0.0.0 |
| 1515 | Wazuh Agent enrollment | 0.0.0.0 |

The Indexer API was listening on the loopback interface. This was recorded as part of the existing architecture rather than treated as a failure, because the assessment did not require direct remote Indexer access.

The Wazuh Manager service status also showed the API process running under the Manager service.

## 3. Configuration Dependency Findings

### Windows Agent Configuration

Configuration file:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

Observed properties:

- File size: 10,059 bytes
- Last modified: 15 October 2026, 06:34:50

The configuration inspection identified references to:

- `<localfile>`
- `<rootcheck>`
- `<sca>`
- `<syscheck>`
- `<active-response>`

These sections indicate that the Agent configuration includes log collection, rootcheck, security configuration assessment, file integrity monitoring, and active-response settings.

The inspection identified the presence of these configuration sections but did not validate every setting against the proposed target version.

### Linux Manager Configuration

The following files were identified:

| File | Observation |
|---|---|
| `/var/ossec/etc/ossec.conf` | Present; approximately 9.1 KB |
| `/var/ossec/etc/rules/local_rules.xml` | Local custom rules file |
| `/var/ossec/etc/rules/local_rules.xml.bak` | Existing backup copy |
| `/var/ossec/etc/rules/local_rules.xml.bak-lab115` | Additional lab backup copy |
| `/var/ossec/etc/decoders/local_decoder.xml` | Local decoder file |
| `/etc/filebeat/filebeat.yml` | Present; 985 bytes |

The presence of local rules and a local decoder is relevant because custom detection logic and parsing behavior should be reviewed before an upgrade.

### Indexer Plugins

The OpenSearch plugin inventory was inspected.

The output included plugins associated with:

- Alerting
- Anomaly detection
- Asynchronous search
- Cross-cluster replication
- Index management
- Machine learning
- Notifications
- Observability
- Security
- SQL

Two plugin-list inspections produced slightly different output. Therefore, the exact plugin inventory should be reconfirmed before final compatibility approval.

A complete plugin compatibility review was not performed.

## 4. Backup and Recovery Evidence

The Windows Agent backup created during Lab 121 was reviewed.

Backup directory:

`C:\Users\Dell\Wazuh-Backup-Lab121\20261029-071243\`

Files identified:

- `ossec.conf`
- `recovery-test\ossec.conf`

Both files were 10,059 bytes.

The backup SHA-256 hash was:

`A3C8010CE63BA05F18972D4EA4582C1B96A0CB40B7052DF6AEBD7FA672A8F72B`

The previous Lab 121 validation established that the original configuration and staged recovery copy matched the backup hash.

This supports the integrity of the backed-up Windows Agent configuration and the staged file-copy recovery test.

The following items were not established:

- A verified backup of the Wazuh Manager configuration and custom rules.
- A verified backup of the Indexer configuration and data.
- A recent, usable Indexer snapshot.
- A complete recovery test of all central Wazuh components.

A search for backup files under `/var/ossec/etc` identified `local_rules.xml.bak`. This is evidence of a local rules backup file, not proof of a complete Manager backup.

## 5. API Connectivity Investigation

The initial API request from Windows to:

`https://192.168.203.129:55000`

failed with a TCP connection timeout.

A subsequent TCP connectivity test showed:

- Ping: Successful
- Source IP: `192.168.203.1`
- Network interface: VMware Network Adapter VMnet8
- TCP port 55000: Failed

The API listener and firewall access were then addressed. The API was subsequently observed listening on port 55000, and the Windows host was able to reach the API endpoint.

A request to `/agents?limit=20` returned:

- Total affected items: 2
- Total failed items: 0

However, a later authentication attempt using the recovered `wazuh` API credentials returned:

`Unauthorized - Invalid credentials`

The subsequent `/manager/info` request failed with:

`Unauthorized - Invalid authorization header`

The latter occurred because a valid token had not been obtained.

The API authentication issue remained unresolved at the end of the recorded investigation.

The installation credentials file was located at:

`/root/wazuh-credentials/wazuh-install-files/wazuh-passwords.txt`

Its contents were exposed during troubleshooting. The affected credentials should be rotated.

## 6. Compatibility Assessment

The observed version baseline was:

| Component | Current Version | Proposed Target |
|---|---|---|
| Wazuh Agent | 4.12.0 | To be evaluated |
| Wazuh Manager | 4.13.1 | 4.14.9 |
| Wazuh Indexer | 4.13.1 | 4.14.9 |
| Wazuh Dashboard | 4.13.1 | 4.14.9 |

The central components were aligned on version 4.13.1.

The Manager was newer than the previously identified Agent version. This is consistent with the general Wazuh version relationship in which the Manager should be the same version as or newer than its Agents. However, the exact compatibility and upgrade sequence must still be checked against the documentation for the proposed target release.

The target version was selected for planning purposes only. The assessment did not verify every target-specific operating system, component, plugin, and configuration requirement.


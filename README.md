# Lab 122 - Wazuh Upgrade Compatibility Assessment

## Overview

This lab documents a pre-upgrade compatibility assessment of a Wazuh deployment. The objective was to establish the current component versions, inspect the deployment architecture and configuration dependencies, review backup readiness, and identify issues that could affect a proposed upgrade.

The assessment focused on identifying potential compatibility concerns before making changes to the environment. No Wazuh upgrade was performed.

## Objectives

- Identify the installed versions of the Wazuh Agent, Manager, Indexer, and Dashboard.
- Establish the current deployment architecture and service status.
- Identify configuration files, custom rules, decoders, and installed plugins.
- Review existing backup and recovery evidence.
- Examine API connectivity and authentication.
- Record compatibility considerations for the proposed target version.
- Document unresolved checks and determine whether the environment is ready for an upgrade.

## Lab Environment

| Component | Environment / Version |
|---|---|
| Endpoint OS | Windows 11 Pro |
| Wazuh Agent | 4.12.0 |
| Server OS | Rocky Linux 9.8 |
| Wazuh Manager | 4.13.1 |
| Wazuh Indexer | 4.13.1 |
| Wazuh Dashboard | 4.13.1 |
| Filebeat | 7.10.2 |
| Proposed target version | 4.14.9 |
| Virtualization | VMware |
| Wazuh VM IP | 192.168.203.129 |
| Windows host VMnet8 IP | 192.168.203.1 |

The proposed target version was selected for assessment purposes. It does not represent an approved upgrade decision.

## Assessment Activities

### 1. Component Version Baseline

The Windows Wazuh Agent version was identified through Windows installed-application metadata.

The Linux central components were identified using RPM package metadata.

The assessment established the following baseline:

| Component | Installed Version |
|---|---|
| Wazuh Agent | 4.12.0 |
| Wazuh Manager | 4.13.1 |
| Wazuh Indexer | 4.13.1 |
| Wazuh Dashboard | 4.13.1 |
| Filebeat | 7.10.2 |

The Manager, Indexer, and Dashboard were aligned on version 4.13.1. The Agent was running an earlier version, 4.12.0.

### 2. Deployment Architecture and Service Status

The Wazuh server was running on Rocky Linux 9.8 under VMware.

The VM used the address `192.168.203.129`, while the Windows host used `192.168.203.1` on the VMnet8 network.

The following services were found active:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

The API listener was observed on port 55000. The Dashboard was listening on port 443, and the Indexer API was listening on port 9200 on the loopback interface.

Agent communication and enrollment listeners were also observed on ports 1514 and 1515.

### 3. Configuration Dependency Inspection

The Windows Agent configuration was located at:

`C:\Program Files (x86)\ossec-agent\ossec.conf`

The file was 10,059 bytes, with a recorded last-modified timestamp of 15 October 2026.

The inspection identified configuration sections associated with:

- Local log collection
- Rootcheck
- Security Configuration Assessment (SCA)
- Syscheck
- Active response

On the Linux server, the following files were identified:

- `/var/ossec/etc/ossec.conf`
- `/var/ossec/etc/rules/local_rules.xml`
- `/var/ossec/etc/decoders/local_decoder.xml`
- `/etc/filebeat/filebeat.yml`

The inspection also identified additional local rule backup files and installed OpenSearch plugins on the Indexer.

These findings established the presence of configuration dependencies that should be reviewed before an upgrade. A complete compatibility validation of every custom setting and plugin was not performed.

### 4. Backup Readiness Review

The Windows Agent configuration backup created during Lab 121 was reviewed.

The following files were present:

- Original backup: `ossec.conf`
- Staged recovery copy: `recovery-test\ossec.conf`

Both files were 10,059 bytes.

The SHA-256 hash of the backup was:

`A3C8010CE63BA05F18972D4EA4582C1B96A0CB40B7052DF6AEBD7FA672A8F72B`

Lab 121 had previously established that the original configuration and staged recovery copy matched the backup hash.

This provided evidence of backup integrity and staged file-copy recovery for the Windows Agent configuration.

However, backups of the central Wazuh components and a usable Indexer snapshot were not verified during this assessment.

### 5. API Connectivity and Authentication

The initial Windows-to-Wazuh API connection attempt to port 55000 failed, even though the VM responded to ping.

The API listener and network configuration were subsequently addressed, and TCP connectivity from the Windows host to the Wazuh VM was restored.

A request to the Agents API endpoint returned two affected agent records with no failed items.

However, a later authentication attempt using credentials recovered from the Wazuh installation archive returned `Invalid credentials`. Consequently, the Manager information endpoint could not be consistently validated using a newly authenticated API session.

The API password information was exposed during troubleshooting and should be rotated.

## Key Findings

1. The three central Wazuh components were aligned on version 4.13.1.
2. The Windows Agent was running version 4.12.0.
3. Filebeat 7.10.2 was installed and running.
4. The deployment was hosted on Rocky Linux 9.8 under VMware.
5. Custom configuration files, local rules, a local decoder, and Indexer plugins were identified.
6. The Windows Agent configuration backup had previously passed SHA-256 integrity and staged recovery validation in Lab 121.
7. Central component backups and Indexer snapshot recovery readiness remained unverified.
8. API TCP connectivity was restored, but API authentication remained unresolved.

## Assessment Outcome

**Current decision: Further investigation required.**

The assessment established a useful version baseline and identified configuration and backup dependencies. However, it did not establish complete upgrade readiness.

Before approving an upgrade, the following work remains:

- Reconfirm the Windows Agent version and its compatibility with the proposed target.
- Review the official upgrade path and component-specific requirements.
- Validate custom rules, decoders, configuration settings, and plugins.
- Verify central component backups and a usable Indexer snapshot.
- Resolve API authentication and rotate exposed credentials.
- Document a recovery plan for the complete Wazuh deployment.

No upgrade, package installation, or component migration was performed as part of this lab.

## References

- [Wazuh Upgrade Guide](https://documentation.wazuh.com/current/upgrade-guide/index.html)
- [Upgrading Wazuh Central Components](https://documentation.wazuh.com/current/upgrade-guide/upgrading-central-components.html)
- [Wazuh Agent Upgrade Guide](https://documentation.wazuh.com/current/upgrade-guide/wazuh-agent/index.html)

## Disclaimer

This is a hands-on lab assessment conducted in a VMware-based environment. The findings represent the observed configuration and test results at the time of assessment. They do not constitute approval for a production upgrade or proof of full disaster-recovery readiness.

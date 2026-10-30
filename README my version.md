# Wazuh-Upgrade-Compatibility-Assessment
## Overview

### Wazuh Upgrade Compatibility Assessment

A Wazuh upgrade compatibility assessment is a pre-upgrade review used to determine whether an existing Wazuh deployment can move to a proposed version safely and what needs to be checked before making changes.

The goal is not to perform the upgrade immediately. It is to understand the current environment, compare it against the target version's requirements, identify potential problems, and decide whether the deployment is ready.

A Wazuh deployment may include several connected components:

- **Wazuh Agent:** Collects security events and endpoint telemetry from monitored systems.
- **Wazuh Manager:** Receives and analyzes agent data and applies detection rules.
- **Wazuh Indexer:** Stores and indexes security data for searching and analysis.
- **Wazuh Dashboard:** Provides the interface for investigating alerts and managing the deployment.

These components have related compatibility requirements. An upgrade assessment should therefore consider the deployment as a whole rather than checking only one component's version.

### Important Concepts

| Concept | Simple Explanation |
|---|---|
| **Version baseline** | Recording the versions currently installed across the deployment. |
| **Target version** | The specific Wazuh release being considered for upgrade. |
| **Compatibility** | Whether the current components and dependencies are supported with the proposed target versions. |
| **Configuration dependencies** | Custom rules, decoders, settings, certificates, integrations, and other components that may be affected. |
| **Backup readiness** | Confirming that appropriate backups exist and that recovery procedures have been tested. |
| **Upgrade risk** | The possibility of service interruption, configuration problems, compatibility failures, or loss of functionality. |
| **Go/no-go decision** | Deciding whether to proceed, investigate further, or postpone the upgrade based on evidence. |

### Example Scenario

Suppose the Wazuh Agent is running version `4.12.0`, and an administrator is considering upgrading to a newer release.

Before approving the upgrade, the analyst:

1. Records the installed versions across all Wazuh components.
2. Identifies the intended target release.
3. Reviews the official Wazuh upgrade documentation.
4. Checks custom configurations, rules, decoders, and integrations.
5. Confirms that appropriate backups exist and recovery procedures have been tested.
6. Documents compatibility concerns and potential upgrade risks.

The findings are then used to determine whether the deployment is ready for an upgrade or requires further investigation.

> **Important:** A newer version is not automatically compatible with every existing component or configuration. Compatibility must be verified against the official documentation for the specific target release.

This lab documents a pre-upgrade compatibility assessment of a Wazuh deployment. The objective was to establish the current component versions, inspect the deployment architecture and configuration dependencies, review backup readiness, and identify issues that could affect a proposed upgrade.

The assessment focused on identifying potential compatibility concerns before making changes to the environment. No Wazuh upgrade was performed.

## Lab Objectives

The objectives of this lab are to:

- Establish a verified version baseline for the Wazuh Agent, Manager, Indexer, Dashboard, and supporting components.
- Examine the relationship between endpoint and central component versions to identify potential compatibility concerns.
- Identify the operating system, deployment architecture, active services, and network listeners supporting the Wazuh environment.
- Inspect existing Agent and Manager configuration files to identify custom settings and dependencies that may require attention during an upgrade.
- Review local detection rules, decoders, Filebeat configuration, and installed Indexer plugins for potential upgrade-related considerations.
- Evaluate existing backup evidence, including configuration file integrity and previously performed staged recovery validation.
- Identify gaps in central component backup coverage and Indexer snapshot readiness.
- Investigate API connectivity and authentication issues that may affect administrative verification.
- Compare the existing deployment against the requirements and upgrade guidance for the proposed target version.
- Document compatibility findings, unresolved dependencies, potential risks, and recovery considerations.
- Establish an evidence-based go/no-go assessment without performing the actual upgrade.

## Lab Scenario

A Wazuh deployment is currently operating with different versions across its endpoint and central components. The Windows endpoint is running Wazuh Agent 4.12.0, while the Wazuh Manager, Indexer, and Dashboard are running version 4.13.1. The environment is hosted on a Rocky Linux 9.8 virtual machine, with the Windows endpoint communicating with the central deployment over a VMware virtual network.

An upgrade to Wazuh 4.14.9 is being considered. Before proceeding, the existing deployment must be assessed to determine whether its components, operating system, configurations, supporting services, and installed dependencies are suitable for the proposed target version.

The assessment focuses on the following areas:

- **Version and compatibility review:** Establish the installed versions of the Agent, Manager, Indexer, Dashboard, and supporting components. Identify version differences and verify the applicable upgrade and compatibility requirements.
- **Configuration dependencies:** Inspect the Windows Agent and Linux Manager configurations, including local rules, decoders, Filebeat settings, and installed Indexer plugins. Identify custom configurations that may require additional validation.
- **Service and network health:** Review the status of Wazuh services, API availability, and active network listeners to understand the current deployment state and identify administrative access issues.
- **Backup and recovery readiness:** Examine existing configuration backups and previously validated recovery copies. Identify gaps in central component backups and determine whether Indexer snapshot readiness has been demonstrated.
- **Upgrade risks:** Document unresolved compatibility questions, API authentication issues, configuration dependencies, and recovery limitations that could affect the upgrade process.

The assessment is intended to establish whether the environment has sufficient evidence and preparation to proceed with an upgrade. Any identified gaps must be documented rather than treated as resolved without verification.

**Scope limitation:** This is a pre-upgrade compatibility and readiness assessment only. No Wazuh component upgrade is performed. The proposed target version is an assessment candidate, not an approved upgrade decision. The final readiness recommendation must be based on verified compatibility requirements, backup coverage, and resolution of outstanding issues.

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


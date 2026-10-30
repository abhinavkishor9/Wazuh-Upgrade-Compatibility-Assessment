
# Troubleshooting Notes - Wazuh Upgrade Compatibility Assessment

## 1. Linux Package Manager Identification

### Issue

The initial package inventory command used `dpkg`:

```bash
dpkg -l | grep -Ei 'wazuh|filebeat'
```

The Linux VM returned:

```text
bash: dpkg: command not found
```

### Cause

The Wazuh server was running Rocky Linux 9.8, which uses the RPM package management system rather than Debian's `dpkg` package manager.

### Resolution

The operating system was identified using:

```bash
cat /etc/os-release
```

Installed packages were then queried using:

```bash
rpm -qa | grep -Ei '^wazuh-(manager|indexer|dashboard)'
```

For a more complete package inventory:

```bash
rpm -q wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

### Outcome

The following versions were identified:

- Wazuh Manager: 4.13.1
- Wazuh Indexer: 4.13.1
- Wazuh Dashboard: 4.13.1
- Filebeat: 7.10.2

The issue was resolved by using the package manager appropriate for the installed Linux distribution.

---

## 2. Incorrect Credentials File Path

### Issue

After extracting the Wazuh installation archive, the following command failed:

```bash
grep -i 'wazuh' /root/wazuh-credentials/wazuh-passwords.txt
```

The system returned:

```text
No such file or directory
```

### Cause

The password file was stored inside a nested directory in the archive:

```text
wazuh-install-files/wazuh-passwords.txt
```

The extraction had succeeded, but the expected file path was incorrect.

### Resolution

The extracted file was located using:

```bash
find /root/wazuh-credentials -type f -name 'wazuh-passwords.txt'
```

The correct path was:

```text
/root/wazuh-credentials/wazuh-install-files/wazuh-passwords.txt
```

### Outcome

The installation credentials file was located, and the API usernames were identified:

- `wazuh`
- `wazuh-wui`

The corresponding password information was present in the file.

However, authentication using the recovered `wazuh` credentials subsequently returned `Invalid credentials`.

The credentials were not successfully validated.

**Security note:** Credential values were exposed during troubleshooting. The affected credentials must be rotated and should not be committed to the repository.

---

## 3. Wazuh API TCP Connectivity Failure

### Issue

The initial PowerShell request to the Wazuh API failed:

```text
A connection attempt failed because the connected party did not properly respond after a period of time.
```

The endpoint was:

```text
https://192.168.203.129:55000
```

### Initial Connectivity Test

The following command was executed:

```powershell
Test-NetConnection 192.168.203.129 -Port 55000
```

The result showed:

- Ping succeeded.
- Source address: `192.168.203.1`
- Interface: VMware Network Adapter VMnet8
- TCP connection to port 55000 failed.

### Investigation

The Linux VM was inspected for:

- API listener status.
- Wazuh Manager service status.
- Firewall configuration.
- VM network address.

The API was subsequently observed listening on:

```text
0.0.0.0:55000
```

The firewall was configured to permit TCP port 55000 from the Windows host address `192.168.203.1`.

### Resolution

The API network access issue was addressed by correcting the listener/firewall access configuration.

The Windows host was subsequently able to establish TCP connectivity to the API port.

### Outcome

TCP connectivity was restored.

However, successful TCP connectivity did not establish successful API authentication.

---

## 4. Invalid API Authorization Header

### Issue

The `/manager/info` request returned:

```json
{
  "title": "Unauthorized",
  "detail": "Invalid authorization header"
}
```

### Cause

A valid bearer token had not been obtained because the preceding authentication request had failed.

The API request was therefore sent with an empty or invalid token.

### Resolution Attempt

The PowerShell authentication request was updated to explicitly use Basic authentication:

```powershell
$Login = Invoke-RestMethod `
    -Method POST `
    -Uri "$BaseUrl/security/user/authenticate" `
    -Authentication Basic `
    -Credential $Credential `
    -SkipCertificateCheck
```

### Outcome

The subsequent authentication attempt still returned:

```text
Unauthorized - Invalid credentials
```

No valid token was obtained, so the Manager information endpoint could not be successfully queried through that authentication attempt.

The authorization-header error was a downstream consequence of failed authentication, not a separate network connectivity failure.

### Follow-up

- Verify the correct API username and password.
- Confirm whether the installation archive credentials are still valid.
- Rotate exposed credentials.
- Use the supported Wazuh password-management procedure if a reset is required.
- Retry authentication before making further API requests.

---

## 5. Indexer Snapshot Verification Incomplete

### Issue

An attempt was made to inspect Indexer snapshot repositories using:

```bash
curl -k -u admin "https://127.0.0"
```

The command returned an empty JSON object:

```json
{}
```

### Observation

The URL shown in the captured command was incomplete. It did not query the Indexer snapshot API endpoint.

Therefore, the returned `{}` does not establish that a snapshot repository exists or that a usable snapshot is available.

### Outcome

Indexer snapshot readiness remained unverified.

### Follow-up

Use the correct Indexer snapshot API endpoint and verify:

- Configured snapshot repositories.
- Available snapshots.
- Snapshot creation timestamps.
- Snapshot state and completion status.
- Whether the required indices and cluster state are included.
- Whether restoration can be performed successfully.

Do not treat the presence of a snapshot repository as proof of a recent, restorable backup.

---

## 6. General Lessons Learned

1. Identify the Linux distribution before selecting package-management commands.
2. Verify extracted file paths instead of assuming archive contents are placed directly in the destination directory.
3. Separate network connectivity failures from API authentication failures.
4. A successful ping does not prove that a specific TCP port is reachable.
5. A successful TCP connection does not prove that API credentials are valid.
6. A bearer-token request must only be made after successful authentication.
7. Keep credentials and installation password archives out of Git repositories.
8. A configuration backup and a full-platform backup are not equivalent.
9. A snapshot repository listing is not proof that a usable snapshot exists.
10. Preserve read-only assessment boundaries until compatibility and recovery checks are complete.

## Final Troubleshooting Status

| Issue | Outcome |
|---|---|
| Debian package command unavailable | Resolved using RPM |
| Credentials file path incorrect | Resolved by locating nested directory |
| API TCP connectivity failure | Restored |
| API authentication failure | Unresolved |
| Invalid bearer authorization header | Explained by missing valid token |
| Indexer snapshot verification | Incomplete |
| Full central component backup validation | Not completed |

No Wazuh upgrade was performed during troubleshooting.

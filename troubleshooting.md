# Troubleshooting — Samba AD & Pi-hole DNS Conflict

## 1. Issue Overview

During the deployment of a Samba Active Directory Domain Controller on Debian 12, I encountered a DNS port conflict with an existing Pi-hole installation.

The Samba AD service failed to start because Pi-hole was already listening on port 53, which is required by Samba's internal DNS server.

**Environment:**

- Operating System: Debian 12
- Hostname: `srvads.luhoop.lan`
- Domain: `luhoop.lan`
- Server IP: `192.168.0.49`
- Remote access: Tailscale VPN
- DNS services: Pi-hole FTL and Samba AD

## 2. Problem Identification

After configuring Samba AD, I checked the service status:

```bash
systemctl status samba-ad-dc
```

The service was in a failed state:

```text
Active: failed (Result: exit-code)
```

The logs showed the following errors:

```text
Failed to listen on 0.0.0.0:53
NT_STATUS_ADDRESS_ALREADY_ASSOCIATED

Failed to bind to :::53 TCP
NT_STATUS_ADDRESS_ALREADY_ASSOCIATED

dns failed to setup interfaces
```

These errors indicated that another process was already using the DNS port.

## 3. Investigation

To identify which service was using port 53, I executed:

```bash
ss -lntup | grep ':53 '
```

The output showed:

```text
0.0.0.0:53  users:(("pihole-FTL",pid=1138))
[::]:53     users:(("pihole-FTL",pid=1138))
```

Pi-hole FTL was listening on port 53 across all IPv4 and IPv6 interfaces.

I also checked the Docker containers:

```bash
docker ps
```

No Pi-hole container was running. Further investigation confirmed that Pi-hole was installed as a systemd service directly on Debian.

## 4. Root Cause

Both Samba AD and Pi-hole require access to DNS port 53.

Pi-hole was already bound to all network interfaces using:

- `0.0.0.0:53` — IPv4
- `[::]:53` — IPv6

As a result, Samba AD could not initialize its internal DNS service.

This prevented the domain controller from starting correctly.

## 5. Temporary Resolution

To release port 53, I stopped the Pi-hole service:

```bash
systemctl stop pihole-FTL.service
```

I then verified its status:

```bash
systemctl status pihole-FTL.service
```

Result:

```text
Active: inactive (dead)
```

This confirmed that Pi-hole had stopped successfully.

However, Samba AD had not yet been verified as operational after this change.

The next step is to confirm that port 53 is available and restart Samba AD.

```bash
ss -lntup | grep ':53 '
systemctl reset-failed samba-ad-dc
systemctl start samba-ad-dc
systemctl status samba-ad-dc --no-pager
```

## 6. DNS Verification

Once Samba AD starts successfully, the following commands will be used to verify DNS functionality:

```bash
dig @192.168.0.49 srvads.luhoop.lan A
```

```bash
dig @192.168.0.49 _ldap._tcp.luhoop.lan SRV
```

These tests will confirm whether the domain controller resolves its hostname and publishes the required Active Directory service records.

## 7. Planned Permanent Solution

The long-term objective is to run Samba AD and Pi-hole without DNS port conflicts.

The planned architecture will separate their DNS listening addresses.

Samba AD will provide authoritative DNS services for the `luhoop.lan` domain, while Pi-hole will continue filtering DNS requests for other network devices.

The final configuration will be documented after successful deployment and testing.

## 8. Lessons Learned

This issue highlighted several important aspects of Linux system administration:

- Two DNS services cannot bind to the same IP address and port simultaneously.
- Existing services should be identified before deploying new infrastructure components.
- The `ss` command is useful for identifying port conflicts.
- `systemctl` helps diagnose service failures and manage system services.
- DNS availability is essential for Active Directory and Kerberos.
- Temporary troubleshooting actions should be distinguished from permanent configuration changes.

**Current status:** Pi-hole stopped successfully. Samba AD startup and DNS verification are pending.
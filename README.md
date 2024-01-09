# Pop!_OS Desktop Hardening Guide

This guide covers security hardening steps for beginner and intermediate Pop!_OS Linux desktop users. [Pop!_OS](https://pop.system76.com) is a Ubuntu-based distribution from System76, focusing on reliability, speed, and security. 

## Introduction

Pop!_OS balances usability with security. However, production deployments require reducing attack surface through:  

- Service hardening
- Disk encryption  
- Access controls
- Frequent software updates
- Application sandboxing

This guide helps harden Pop!_OS desktops by covering those key areas.  

**Target Audience**: Linux beginners to intermediate administrators securing desktop systems.

**Contents**:

- [System Updates](#system-updates)
- [User Accounts](#user-accounts)
- [Service Hardening](#service-hardening) 
- [Network Hardening](#network-hardening)
- [Disk Encryption](#disk-encryption)
- [Additional Hardening](#additional-hardening) 
- [General Tips](#general-tips)
- [Contributing](#contributing)

## System Updates  

Keep all software updated:

```
sudo apt update  
sudo apt dist-upgrade
```

- Regularly review update logs to understand changes and potential issues.
- Check [Pop!_OS site](https://pop.system76.com/) weekly for updates   
- Backup user data before major OS upgrades  
- Reboot after kernel updates
- If upgrade errors, see [Pop!_OS forum](https://chat.pop-os.org/landing#/) or ask the community  

Enable automatic security updates:  

```
sudo dpkg-reconfigure -p low unattended-upgrades  
```

Related Tutorials:

- [Backing Up Your System](https://support.system76.com/articles/backup-files)
- [Updating System Firmware](https://support.system76.com/articles/system-firmware)



Keep accurate time using NTS (Network Time Security):   

```
# Review for at least 4 NTS peers, no clear IPs  
curl -o /tmp/chrony.conf https://raw.githubusercontent.com/GrapheneOS/infrastructure/main/chrony.conf  

# Apply after inspection passes  
sudo cp /etc/chrony.conf /etc/chrony.conf.orig 
sudo curl -o /etc/chrony.conf https://raw.githubusercontent.com/GrapheneOS/infrastructure/main/chrony.conf   

# Verify with 4+ sources prefixed by *
sudo systemctl restart chrony  
chronyc sourcestats   
```



## User Accounts   

Enforce strong password policies:  

```
sudo apt install libpam-pwquality  
sudo pam-auth-update --enable remember=5 rounds=65536
```

- 16+ characters, reuse after 5 passwords  
- Increase computation cost for cracking   
- Use 2FA like [ente auth](https://github.com/ente-io/auth)

Audit and reduce excessive permissions:   

```
sudo grep -vE "^(#|$)" /etc/group | cut -d: -f1 | sort -u | less   
```

*Review user group assignments closely*   

Auto-logout after 10 mins inactive:   

```  
sudo nano /etc/lightdm/lightdm.conf
[Seat:*] 
autologin-user=
autologin-session=  
autologin-user-timeout=600
```  

Audit and remove unneeded accounts.

For remote access, set up passwordless SSH authentication using public keys instead of password authentication.

## Service Hardening   

**Unnecessary Services**: Debug, unused hardware, obsolete protocols  
**Examples of Unnecessary Services**: Bluetooth, printing, sound, Thunderbolt, debug logging, SNMP, NFS


Disable services:

```
sudo systemctl list-unit-files --state=enabled 
sudo systemctl disable <service>
sudo systemctl disable bluetooth.service cups.service pulseaudio.service
```

Prevent restarting:  

```
sudo systemctl mask <service>   
``` 

*Test changes safely before system-wide rollout*

## Network Hardening   

Employ firewall to filter access:

```
sudo ufw enable 
sudo ufw default deny incoming  
sudo ufw default allow outgoing
```

Common Unnecessary Open Ports: NETBIOS - 139, SNMP - 161, mDNS - 5353

Limit exposed ports:  

```
sudo nmap localhost  
sudo ufw deny <unneeded_port>
sudo ufw deny 139
sudo ufw deny 161 
sudo ufw deny 5353
```  

When on untrusted networks, use a commercial VPN with:

- Strict no-logs policy
- Strong data encryption   
- Leak protection, custom DNS, etc

Related Resources:

- [Choosing a VPN Service](https://www.privacyguides.org/en/vpn/#criteria) 
- [Port Scanning with Nmap](https://nmap.org/book/port-scanning-tutorial.html)

## Disk Encryption   

Use LUKS to encrypt sensitive data:

``` 
sudo apt install cryptsetup
sudo cryptsetup luksFormat /dev/<disk>  
sudo cryptsetup luksOpen /dev/<disk> name
```
  
- Can noticeably lower disk performance 
- Backup data before enabling encryption
- Record passphrases/keys offline  

For user data, create an encrypted home partition separate from the OS:
```
sudo cryptsetup luksFormat /dev/<home_partition>
```

Consider performance impacts and recovery strategies for encrypted data.

*Irrecoverable if encryption keys are lost*  


Related Resource:
  
- [Pop!_OS Disk Encryption](https://support.system76.com/articles/advanced-luks)


## Additional Hardening  

- **Important Note on Secure Boot**
> 
> As of the current release, Pop!_OS does not support Secure Boot. Enabling Secure Boot may interfere with the boot process, leading to potential issues with accessing the BIOS setup.
> 
> **Recommendation**:
> - Users should **disable Secure Boot** when using Pop!_OS to ensure a smooth operating experience.
> - For the most up-to-date information and detailed instructions, please refer to [System76's official documentation on installing Pop!_OS](https://support.system76.com/articles/install-pop).

- Use application sandboxing tools like Firejail.
- Install security tools like antivirus, IDS.
- Antivirus: ClamAV (opensource antivirus engine for detecting various malicious threats. It's a standard choice for Linux users due to its effectiveness and flexibility)

#### Installation:

```bash
sudo apt install clamav clamav-daemon
```

#### Running a Scan:

Execute a recursive scan with:

```bash
sudo clamscan -r /path/to/scan
```

#### Automating Virus Definitions Updates:

Enable automatic updates for virus definitions:

```bash
sudo systemctl enable clamav-freshclam.service
```

#### Considerations:

- Schedule scans during low-usage times to minimize impact on system performance.
- Regularly review scan logs for potential threats or false positives.
------------------------------------------------------------------------------------------

- Check logs/alerts for intrusion signs.
- Keep system and firmware updated.
- Perform security audits/training
- Consider hardware security features like TPMs.
- Refine BIOS/UEFI settings for security.
- Manage user privileges through sudoers configuration for refined access control.

Auditing Tools: Lynis, CIS-CAT Benchmark
```
sudo apt install lynis
lynis audit system
```

## General Tips 

- Avoid running as root, use `sudo` for privileges  
- Practice safe web browsing habits
- Use VPNs/firewalls on public networks 
- Backup data regularly and store offline  
- Encrypt disks and enable full disk encryption  

## Contributing  

To suggest improvements:
- Open a clearly documented issue/PR
- Follow Python style guides and test contributions  
- Use commit messages like: "$Area: Implement $feature"

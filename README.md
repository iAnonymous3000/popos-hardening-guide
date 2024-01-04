# Pop!_OS Desktop Hardening Guide

This guide covers security hardening steps for beginner and intermediate Pop!_OS Linux desktop users. [Pop!_OS](https://pop.system76.com) is an Ubuntu-based distribution from System76, focusing on reliability, speed, and security. 

## Introduction

Pop!_OS balances usability with security. However, production deployments require reducing attack surface through:  

- Service hardening
- Disk encryption  
- Access controls
- Frequent software updates

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

## Service Hardening   

**Unnecessary Services**: Debug, unused hardware, obsolete protocols  

Disable services:

```
sudo systemctl list-unit-files --state=enabled 
sudo systemctl disable <service>
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

Limit exposed ports:  

```
sudo nmap localhost  
sudo ufw deny <unneeded_port>
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

Related Resource:
  
- [Pop!_OS Disk Encryption](https://support.system76.com/articles/advanced-luks)

*Irrecoverable if encryption keys are lost*  

## Additional Hardening  

- Enable Secure Boot for boot validation 
- Install security tools like antivirus, IDS 
- Check logs/alerts for intrusion signs
- Keep system and firmware updated  
- Perform security audits/training

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

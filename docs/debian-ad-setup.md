# Joining Debian to Active Directory

This document describes how to join a Debian system to an Active Directory (AD) domain.

## Install Required Packages

1. Update package lists:
   ```bash
   sudo apt update
   ```
2. Install realmd and sssd along with common tools:
   ```bash
   sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
   ```

## Discover the Domain

Use `realm discover` to verify that the domain can be found:
```bash
realm discover example.com
```
Replace `example.com` with your AD domain.

## Join the Domain

Join using `realm join` with an account that has rights to add computers:
```bash
sudo realm join -U Administrator example.com
```
During the join process you will be prompted for the account password.

## Configure SSSD

Edit `/etc/sssd/sssd.conf` so that it includes the domain section:
```ini
[sssd]
services = nss, pam
config_file_version = 2
domains = example.com

[domain/example.com]
ad_domain = example.com
id_provider = ad
auth_provider = ad
access_provider = ad
ldap_sudo_search_base = ou=Sudoers,dc=example,dc=com
```
Set permissions:
```bash
sudo chmod 600 /etc/sssd/sssd.conf
```
Restart SSSD:
```bash
sudo systemctl restart sssd
```

## Update PAM and NSS

Ensure NSS and PAM are configured to use SSSD:
```bash
sudo pam-auth-update --enable mkhomedir
```
Verify `/etc/nsswitch.conf` contains `sss` entries for `passwd`, `group` and `shadow`.

## Testing

1. Attempt to list domain users:
   ```bash
   getent passwd "DOMAIN\\username"
   ```
2. Log in with a domain account either locally or over SSH.
3. Verify that the home directory is created and permissions are correct.

## Automation Hints

* Wrap the above steps into an Ansible playbook using the `realm` and `sssd_conf` modules.
* Use cloud-init or shell scripts to automate package installation and domain join during provisioning.


# Install Kerberos (Standalone KDC) for Cloudera

This repository contains step-by-step instructions to install and configure a **Kerberos KDC + Admin Server** (standalone) for integration with **Cloudera Manager** and **CDP Runtime**.

## Contents

- [`install_kerberos.md`](install_kerberos.md) → full installation & configuration guide
- Example configs:
  - `/etc/krb5.conf`
  - `/var/kerberos/krb5kdc/kdc.conf`
  - `/var/kerberos/krb5kdc/kadm5.acl`

## Features

- Standalone KDC + Admin Server setup
- Secure keytab generation for Cloudera Manager
- Time sync with chrony
- Verification steps (kinit, klist)
- Integration with Cloudera Manager (enable Kerberos in CM UI)

## Requirements

- RHEL / CentOS 7 or 8
- Root privileges
- NTP/chrony installed and running
- Database already prepared for Cloudera Manager


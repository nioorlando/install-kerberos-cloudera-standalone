# 📘 Install & Configure Kerberos (Standalone KDC)

Step-by-step installation of a **Kerberos KDC + Admin Server** for integration with **Cloudera Manager**.

---

## 1. Install Kerberos Packages
```bash
sudo yum install -y krb5-server krb5-libs krb5-workstation
```

---

## 2. Configure `/etc/krb5.conf`
```ini
includedir /etc/krb5.conf.d/

[logging]
 default      = FILE:/var/log/krb5libs.log
 kdc          = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 default_realm    = (REALM)
 dns_lookup_realm = false
 dns_lookup_kdc   = false
 ticket_lifetime  = 24h
 renew_lifetime   = 7d
 forwardable      = true
 rdns             = false

[realms]
 (REALM) = {
   kdc          = (hostname)
   admin_server = (hostname)
 }

[domain_realm]
 .(domain) = (REALM)
 (domain)  = (REALM)
```

---

## 3. Configure `/var/kerberos/krb5kdc/kdc.conf`
```ini
[kdcdefaults]
 kdc_ports     = 88
 kdc_tcp_ports = 88

[realms]
 (REALM) = {
   acl_file     = /var/kerberos/krb5kdc/kadm5.acl
   dict_file    = /usr/share/dict/words
   admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
   supported_enctypes = rc4-hmac:normal aes256-cts:normal aes128-cts:normal
   max_life            = 24h
   max_renewable_life  = 7d
 }
```

---

## 4. Configure `/var/kerberos/krb5kdc/kadm5.acl`
```text
admin@(REALM)    *
```

---

## 5. Initialize KDC Database
```bash
sudo kdb5_util create -s
# enter (master_password)
```

---

## 6. Create Admin Principal
```bash
sudo kadmin.local
addprinc admin
# enter (password)
exit
```

---

## 7. Generate Keytab for Cloudera Manager
```bash
sudo kadmin.local
ktadd -k /var/run/cloudera-scm-server/admin.keytab admin@(REALM)
exit

sudo chown cloudera-scm:cloudera-scm /var/run/cloudera-scm-server/admin.keytab
sudo chmod 400 /var/run/cloudera-scm-server/admin.keytab
```

---

## 8. Enable & Start KDC Services
```bash
systemctl enable --now krb5kdc
systemctl enable --now kadmin
```

---

## 9. Sync Time with Chrony
```bash
systemctl enable --now chronyd
chronyc makestep
chronyc tracking
```

---

## 10. Verify KDC
```bash
telnet localhost 88
telnet localhost 749
```

Test login:
```bash
kinit admin
# enter (password)
klist
```

---

## 11. Configure Cloudera Manager for Kerberos

- Kerberos Realm: `(REALM)`  
- KDC Server Host: `(hostname)`  
- Admin Server Host: `(hostname)`  
- Encryption types: rc4-hmac, aes128-cts, aes256-cts  
- krb5.conf path: `/etc/krb5.conf`  

Restart CM services:
```bash
systemctl restart cloudera-scm-server
systemctl restart cloudera-scm-agent
```

Enable Kerberos in CM UI → Generate Missing Credentials.

---

## 12. Troubleshooting

- **Generate credentials failed** → re-create keytab, check permission 400  
- **Password mismatch** → reset in KDC & update in CM  
- **Privilege error** → ensure `kadm5.acl` has `admin@(REALM) *`  
- **Time skew** → fix with chrony  

---

## 13. Final Test
```bash
kinit admin
hdfs dfs -ls /
```

Expected: HDFS listing works with Kerberos authentication.

---

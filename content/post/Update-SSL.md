---
date: '2026-01-01T16:17:48+07:00'
draft: false
title: "Update SSL"
ShowToc: true
---

## Update SSL 
note❗: This is an example for server that uses the httpd service listens on port 443.

### Action to check expired certificates
```
site="<ip-server>" ; port="443" ;echo | openssl s_client -servername $site -connect $site:$port | openssl x509 -noout -dates
```

### SCP Cert from Local to Target
```
scp -r /path/to/source/ user@<server>:/path/to/target
```

### Create Backup Cert
Note ❗: In this example, the SSL httpd service configuration uses the certificate in /etc/ssl/.
```
cd /etc/ssl
mkdir ssl_backup
cp -pfv <current-cert> ssl_backup/
```

### Change the Old Certificate to the New One
Note❗: The source path is taken from the previous SCP result.
```
sudo cp -v path/to/source /etc/ssl/
```

### Check SSL expiration in the target server 
```
openssl x509 -noout -in pegadaian.pem -dates
openssl x509 -noout -in pegadaian.pem -text

(for cert java)
keytool -list -v -keystore pegadaian.jks | less
```

### Restart or Reload Service
```
systemctl restart httpd.service

-or-

systemctl reload httpd.service
```


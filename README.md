# Modul300

## Apache2

### Virtual-Host in /etc/apache2/sites-available

```
<VirtualHost *:80>
ServerName ku1.smartlearn.lan
ServerAlias www.ku1.smartlearn.lan
DocumentRoot /www/ku1.smartlearn.lan
CustomLog /var/log/apache2/ku1.smartlearn.lan-acces.log combined
ErrorLog /var/log/apache2/ku1.smartlearn.lan-error.log
ErrorDocument 404 /err/404.html
</VirtualHost>
```

### Directory /www in /etc/apache2/apache.conf aktivieren

```
<Directory /www>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

### Virtualhost Commands

`a2ensite name` -> Aktiviert den Virtualhost

`a2dissite name` -> Deaktiviert den Virtualhost (000-default deaktivieren wenn Probleme mit Default-Seite)

`systemctl reload apache2` -> Lädt die apache2-Konfig neu

`systemctl restart apache2` -> Startet apache2 neu

## Bind

### Reverse lookup zone (/etc/bind/db.210.168.192)

```
@  IN   SOA (
        ns.smartlearn.dmz.
        root.smartlearn.dmz.
        2021021701
        3600           ; refresh period
        600                        ; retry period
        604800                     ; expire time
        1800                     ) ; minimum ttl
@       IN      NS      ns.smartlearn.dmz.
30.210.168.192.in-addr.arpa. IN  PTR  vmlp1.
1.210.168.192.in-addr.arpa.  IN  PTR  vmlf1.
```

### Forward lookup zone (/etc/bind/db.smartlearn.lan)

```
@  IN	SOA (
	ns.smartlearn.dmz.
	root.smartlearn.dmz.
	2021021701
	3600           ; refresh period
        600                        ; retry period
        604800                     ; expire time
        1800                     ) ; minimum ttl
@	IN	NS	ns.smartlearn.dmz.
vmlp1	IN	A	192.168.210.30
vmlf1	IN	A	192.168.210.1

vmls5	IN	A	192.168.210.61
www	IN	A	192.168.210.61
ftp	IN	A	192.168.210.61
ku1	IN	A	192.168.210.61
ku2	IN	A	192.168.210.61

vmls4	IN	A	192.168.210.60
```

### Enable Zone (/etc/bind/named.conf.local)

```
zone "smartlearn.lan"  {
 type master;
  file "/etc/bind/db.smartlearn.lan";};

zone "210.168.192.in-addr.arpa" {
 type master;
 file "/etc/bind/db.210.168.192";};

zone "smartlearn.dmz" {
 type master;
 file "/etc/bind/db.smartlearn.dmz";
};

zone "220.168.192.in-addr.arpa" {
 type master;
 file "/etc/bind/db.220.168.192";
};
```

### Set Forward DNS Server (/etc/bind/named.conf.options)

```
options {
	directory "/var/cache/bind";
	forwarders { 1.1.1.1;8.8.8.8; };
	allow-query { 192.168.0.0/16;127.0.0.0/8; };
};
```

### Bind commands

`systemctl restart bind` -> Bind neustarten

`tail -f /var/log/syslog` -> Syslog tailen um Fehler zu finden

`systemctl status bind` -> Status des Bind-Service anschauen um Fehler zu finden

`systemd-resolve --set-dns 192.168.220.12 --interface eth0` -> DNS auf DNS-Server ändern, wenn es schnell gehen soll geht auch `nameserver 192.168.220.12` in /etc/resolv.conf. Dies wird nach einem Reboot aber überschrieben.

## ProFTPD

### FTP-Config in /etc/proftpd/proftpd.conf

```
DefaultRoot /www/www.smartlearn.dmz/
AuthOrder mod_auth_file.c
AuthUserFile /etc/proftpd/ftpd.passwd
AuthPam off
RequireValidShell off
```

### Neuer FTP-User erstellen

`ftpasswd --passwd --name=ftpuser --uid=1001 --home=/www --shell=/bin/false` Dies erstellt ein neues User-File namens ftpd.passwd. Hinweis: Dieses File wird am Ort erstellt, wo der Command ausgeführt wird. Somit sollte der Command in /etc/proftpd ausgeführt werden oder das File verschoben werden oder im /etc/proftpd/proftpd.conf der entsprechende Pfad eingetragen werden.

### FTP testen

Mit `ftp ftpuser@www.smartlearn.dmz` auf den FTP-Server einloggen und mit ls testen ob die Dateien vorhanden sind. Auch andere Unix-User testen, diese sollten in diesem Setup nicht gehen.

## Neue Disk Partitionieren

1. Mit `lsblk` die verfügbaren Disken auflisten
2. `fdisk` für die entsprechende Disk starten -> Bsp `fdisk /dev/sdb`
3. `n` für neu auswählen, die default Werte 4 mal mit `Enter` bestätigen, danach mit `w` speichern.
4. Mit `mkfs.ext4 /dev/sdb1` die neu erstellte Partition formatieren.
5. Mit `mount /dev/sdb1 /www` die Disk temporär mounten oder `/dev/sdb1 /www ext4 defaults 0 2` in `/etc/fstab` eintragen und mit `mount -av` testen. Dies sollte gemacht werden, da sonst bei einer Fehlkonfiguration die VM unter Umständen nicht mehr richtig booten kann!

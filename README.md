# Инициализация системы. Systemd
Для выполнения этого действия требуется установить приложением git:
`git clone https://github.com/altyn-kenzhebaev/systemd-hw8.git`
В текущей директории появится папка с именем репозитория. В данном случае systemd-hw8.git. Ознакомимся с содержимым:
```
cd systemd-hw8.git
ls -l
README.md
Vagrantfile
```
Здесь:
- README.md - файл с данным руководством
- Vagrantfile - файл описывающий виртуальную инфраструктуру для `Vagrant`
### Watchlog
Длā начала создаём файл с конфигурацией длā сервиса в директории:
```
vi /etc/sysconfig/watchlog
# Configuration file for my watchlog service
# Place it to /etc/sysconfig

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение:
```
echo ALERT > /var/log/watchlog.log
```
Создадим скрипт:
```
vi /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
    logger "$DATE: I found word, Master!"
else
    exit 0
fi

chmod +x /opt/watchlog.sh
```
Создадим unit для сервиса:
```
vi /etc/systemd/system/watchlog.service
####
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
####
```
Создадим unit для таймера:
```
vi /etc/systemd/system/watchlog.timer
####
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
####
```
Затем достаточно перезапустить systemd, стартануть и убедиться в результате:
```
systemctl daemon-reload
systemctl start watchlog watchlog.timer
[root@systemd ~]# tail -f /var/log/messages 
Mar 31 13:53:16 systemd systemd: Started My watchlog service.
Mar 31 13:53:46 systemd systemd: Starting My watchlog service...
Mar 31 13:53:46 systemd root: Fri Mar 31 13:53:46 +06 2023: I found word, Master!
Mar 31 13:53:46 systemd systemd: Started My watchlog service.
Mar 31 13:54:16 systemd systemd: Starting My watchlog service...
Mar 31 13:54:16 systemd root: Fri Mar 31 13:54:16 +06 2023: I found word, Master!
Mar 31 13:54:16 systemd systemd: Started My watchlog service.
Mar 31 13:54:46 systemd systemd: Starting My watchlog service...
Mar 31 13:54:46 systemd root: Fri Mar 31 13:54:46 +06 2023: I found word, Master!
Mar 31 13:54:46 systemd systemd: Started My watchlog service.
```
### spawn-fcgi systemd.service
Устанавливаем spawn-fcgi и необходимые длā него пакеты:
```
yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```
Далее необходимо раскомментировать строки с переменными:
```
vi /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```
Далее добавляем юнит файл. Он будет примерно следующего вида:
```
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```
Затем достаточно перезапустить systemd, стартануть и убедиться в результате:
```
systemctl daemon-reload
systemctl start spawm-fcgi
systemctl status spawm-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-03-31 12:21:37 +06; 1h 38min ago
 Main PID: 1338 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─1338 /usr/bin/php-cgi
           ├─1339 /usr/bin/php-cgi
           ├─1340 /usr/bin/php-cgi
           ├─1341 /usr/bin/php-cgi
           ├─1342 /usr/bin/php-cgi
           ├─1343 /usr/bin/php-cgi
           ├─1344 /usr/bin/php-cgi
           ├─1345 /usr/bin/php-cgi
           ├─1346 /usr/bin/php-cgi
           ├─1347 /usr/bin/php-cgi
           ├─1348 /usr/bin/php-cgi
           ├─1349 /usr/bin/php-cgi
           ├─1350 /usr/bin/php-cgi
           ├─1351 /usr/bin/php-cgi
           ├─1352 /usr/bin/php-cgi
           ├─1353 /usr/bin/php-cgi
           ├─1354 /usr/bin/php-cgi
           ├─1355 /usr/bin/php-cgi
           ├─1356 /usr/bin/php-cgi
           ├─1357 /usr/bin/php-cgi
           ├─1358 /usr/bin/php-cgi
           ├─1359 /usr/bin/php-cgi
           ├─1360 /usr/bin/php-cgi
           ├─1361 /usr/bin/php-cgi
           ├─1362 /usr/bin/php-cgi
           ├─1363 /usr/bin/php-cgi
           ├─1364 /usr/bin/php-cgi
           ├─1365 /usr/bin/php-cgi
           ├─1366 /usr/bin/php-cgi
           ├─1367 /usr/bin/php-cgi
           ├─1368 /usr/bin/php-cgi
           ├─1369 /usr/bin/php-cgi
           └─1370 /usr/bin/php-cgi

Mar 31 12:21:37 systemd systemd[1]: Started Spawn-fcgi startup service by Otus.
```
### httpd for 3 instances
Для этого потребуется использовать кофигурацию httpd.service как шаблон (/usr/lib/systemd/system/httpd.service):
```
cp /usr/lib/systemd/system/httpd.service /usr/lib/systemd/system/httpd@first.service
vi /usr/lib/systemd/system/httpd@first.service
###
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
Environment=LANG=C
EnvironmentFile=/etc/sysconfig/httpd-%I
EnvironmentFile=/etc/sysconfig/httpd
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
# We want systemd to give httpd some time to finish gracefully, but still want
# it to kill httpd after TimeoutStopSec if something went wrong during the
# graceful stop. Normally, Systemd sends SIGTERM signal right after the
# ExecStop, which would kill httpd. We are sending useless SIGCONT here to give
# httpd time to finish.
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
###

cp /usr/lib/systemd/system/httpd@first.service /usr/lib/systemd/system/httpd@second.service
```
Задаем отдельные конфигурационные файлы:
```
cat /etc/sysconfig/httpd-*
# /etc/sysconfig/httpd-first
OPTIONS=-f conf/first.conf
# /etc/sysconfig/httpd-second
OPTIONS=-f conf/second.conf

cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf
```
В файлах `/etc/httpd/conf/first.conf`, `/etc/httpd/conf/second.conf` добавляем\меняем данные параметры:
```
# /etc/httpd/conf/first.conf
PidFile /var/run/httpd-first.pid
Listen 8080
# /etc/httpd/conf/second.conf
PidFile /var/run/httpd-second.pid
Listen 9090
```
Затем достаточно перезапустить systemd, стартануть и убедиться в результате:
```
systemctl daemon-reload
systemctl start httpd httpd@first.service httpd@second.service
ss -tnulp | grep httpd
tcp    LISTEN     0      128    [::]:80                 [::]:*                   users:(("httpd",pid=21331,fd=4),("httpd",pid=21330,fd=4),("httpd",pid=21329,fd=4),("httpd",pid=21328,fd=4),("httpd",pid=21327,fd=4),("httpd",pid=21326,fd=4))
tcp    LISTEN     0      128    [::]:8080               [::]:*                   users:(("httpd",pid=21247,fd=4),("httpd",pid=21246,fd=4),("httpd",pid=21245,fd=4),("httpd",pid=21244,fd=4),("httpd",pid=21243,fd=4),("httpd",pid=21242,fd=4))
tcp    LISTEN     0      128    [::]:9090               [::]:*                   users:(("httpd",pid=21235,fd=4),("httpd",pid=21234,fd=4),("httpd",pid=21233,fd=4),("httpd",pid=21232,fd=4),("httpd",pid=21231,fd=4),("httpd",pid=21230,fd=4))
```
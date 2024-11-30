# Домашнее задание к занятию "`Защита сети`" - `Дедюрин Денис`

---
### Подготовка к выполнению заданий

Подготовка защищаемой системы:
установите Suricata,
установите Fail2Ban.
Подготовка системы злоумышленника: установите nmap и thc-hydra либо скачайте и установите Kali linux.
Обе системы должны находится в одной подсети.

---
## Задание 1
Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

```
sudo nmap -sA < ip-адрес >

sudo nmap -sT < ip-адрес >

sudo nmap -sS < ip-адрес >

sudo nmap -sV < ip-адрес >
```
По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.

В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат.

### Ответ:

Для выполнения заданий были использованы 2 ВМ:

Машина "взломщик" с установленной ОС Kali-Linux 2024.3 (192.168.1.140)

Машина "жертва" с установленной ОС Centos 9 Stream (192.168.1.106)

Для установки на "жертву" утилит Suricata и Fail2ban выполняем следующие команды:
```
dnf install epel-release -y
dnf install suricate fail2ban -y
```
Пред запуском Suricata произведем первоначальные настройки.

В конфигурационном файле /etc/suricata/suricata.yaml изменим параметры:
```
EXTERNAL_NET: "any"
```
и укажем актуальный интерфейс на котором будет "прослушиваться" активность сети
```
af-packet:
  - interface: enp0s3
```
Далее необходимо выполнить обновление правил:
```
suricata-update
```
После чего запускаем командой:
```
systemctl start suricata
```
Проверяем что все запустилось:
```
systemctl status suricata
```
```
● suricata.service - Suricata Intrusion Detection Service
     Loaded: loaded (/usr/lib/systemd/system/suricata.service; enabled; preset: disabled)
     Active: active (running) since Sat 2024-11-30 15:27:45 MSK; 3s ago
       Docs: man:suricata(1)
    Process: 4558 ExecStartPre=/bin/rm -f /var/run/suricata.pid (code=exited, status=0/SUCCESS)
   Main PID: 4560 (Suricata-Main)
      Tasks: 1 (limit: 23139)
     Memory: 37.1M
        CPU: 3.071s
     CGroup: /system.slice/suricata.service
             └─4560 /sbin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid -i enp0s3 --user suricata

Nov 30 15:27:45 localhost.localdomain systemd[1]: Starting Suricata Intrusion Detection Service...
Nov 30 15:27:45 localhost.localdomain systemd[1]: Started Suricata Intrusion Detection Service.
Nov 30 15:27:45 localhost.localdomain suricata[4560]: i: suricata: This is Suricata version 7.0.7 RELEASE running in SYSTEM mode
```

Дополнительных настроек для Fail2Ban не требуется.

Производим разведку командами:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sA 192.168.1.106
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 07:40 EST
Nmap scan report for 192.168.1.106
Host is up (0.0076s latency).
All 1000 scanned ports on 192.168.1.106 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
MAC Address: 08:00:27:B1:55:F9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds
```
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sT 192.168.1.106
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 07:40 EST
Nmap scan report for 192.168.1.106
Host is up (0.024s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:B1:55:F9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.33 seconds
```
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS 192.168.1.106
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 07:41 EST
Nmap scan report for 192.168.1.106
Host is up (0.0080s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 08:00:27:B1:55:F9 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.51 seconds
```
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV 192.168.1.106
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-30 07:41 EST
Nmap scan report for 192.168.1.106
Host is up (0.018s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.7 (protocol 2.0)
MAC Address: 08:00:27:B1:55:F9 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.62 seconds
```
В логах видно, что suricata зафиксировала попытки сканирования портов с ВМ с адресом 192.168.1.140
```
11/30/2024-15:47:04.992773  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:51818 -> 192.168.1.106:3306
11/30/2024-15:47:05.058290  [**] [1:2002911:6] ET SCAN Potential VNC Scan 5900-5920 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.140:49198 -> 192.168.1.106:5903
11/30/2024-15:47:05.116003  [**] [1:2002910:6] ET SCAN Potential VNC Scan 5800-5820 [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.140:45170 -> 192.168.1.106:5811
11/30/2024-15:47:05.166105  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:52614 -> 192.168.1.106:5432
11/30/2024-15:47:05.167554  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:50376 -> 192.168.1.106:1521
11/30/2024-15:47:05.187345  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:34670 -> 192.168.1.106:1433
11/30/2024-15:47:32.137879  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:46371 -> 192.168.1.106:3306
11/30/2024-15:47:32.206907  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:46371 -> 192.168.1.106:1521
11/30/2024-15:47:32.254010  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:46371 -> 192.168.1.106:5432
11/30/2024-15:47:32.431391  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:46371 -> 192.168.1.106:1433
11/30/2024-15:47:45.671968  [**] [1:2010937:3] ET SCAN Suspicious inbound to mySQL port 3306 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:41325 -> 192.168.1.106:3306
11/30/2024-15:47:45.712109  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:41325 -> 192.168.1.106:5432
11/30/2024-15:47:45.784861  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:41325 -> 192.168.1.106:1433
11/30/2024-15:47:45.794040  [**] [1:2010936:3] ET SCAN Suspicious inbound to Oracle SQL port 1521 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.1.140:41325 -> 192.168.1.106:1521
11/30/2024-15:47:46.109459  [**] [1:2054407:1] ET INFO Server Responded with Vulnerable OpenSSH Version (CVE-2024-6409) [**] [Classification: Large Scale Information Leak] [Priority: 2] {TCP} 192.168.1.106:22 -> 192.168.1.140:43248
```

---
## Задание 2
Проведите атаку на подбор пароля для службы SSH:

```
hydra -L users.txt -P pass.txt < ip-адрес > ssh
``
Настройка hydra:
создайте два файла: users.txt и pass.txt;
в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.
Дополнительная информация по hydra: https://kali.tools/?p=1847.

Включение защиты SSH для Fail2Ban:
открыть файл /etc/fail2ban/jail.conf,
найти секцию ssh,
установить enabled в true.
Дополнительная информация по Fail2Ban:https://putty.org.ru/articles/fail2ban-ssh.html.

В качестве ответа пришлите события, которые попали в логи Suricata и Fail2Ban, прокомментируйте результат

### Ответ
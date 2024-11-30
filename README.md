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
Для выполнения заданий были использованы 2 ВМ
Машина взломщик с установленной ОС Kali-Linux 2024.3
Машина жертва с установленной ОС Centos 9 Stream
---

После установки на "жертву" Suricata произведем первоначальные настройки:

В конфигурационном файле /etc/suricata/suricata.yaml изменим параметры:
```
EXTERNAL_NET: "any"
```
и укажем актуальный интерфейс на котором будет "прослушиваться" активность сети
```
af-packet:
  - interface: enp0s3
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
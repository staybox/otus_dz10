# OTUS ДЗ 10 Пользователи и группы. Авторизация и аутентификация  (Centos 7)
-----------------------------------------------------------------------
### Домашнее задание

    1. Запретить всем пользователям, кроме группы admin логин в выходные(суббота и воскресенье), без учета праздников
    2. Дать конкретному пользователю права рута

1. Запретить всем пользователям, кроме группы admin логин в выходные(суббота и воскресенье), без учета праздников
- Заходим в файл ```nano /etc/pam.d/sshd``` и приводим его к следующему виду:
```
[root@lvm system]# cat /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
account required pam_time.so # Добавляем вот эту строку
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```
- Далее заходим в файл ```nano /etc/security/time.conf``` и добавляем в конце файла строку ```*;*;test_user;!Tu```
- Создаем пользователя командой ```useradd test_user``` и задаем ему пароль ```passwd test_user```
- Пытаемся зайди в тот день, когда у нас работает правило ```ssh test_user@localhost``` и получаем:
```
[root@lvm system]# ssh test_user@localhost
test_user@localhost's password:
Authentication failed.
```
2. Дать конкретному пользователю права рута:
- ```grep test_user /etc/passwd``` - Ищем нашего пользователя, которому надо дать права root
```
[root@lvm system]# grep test_ /etc/passwd
test_user:x:1001:1001::/home/test_user:/bin/bash
```
- Далее меняем два значения ```1001``` на ```0```(значения uid и gid root)
Теперь когда пользователь будет заходить по своему логину, у него будут права рута
- Также есть второй способ, командой ```usermod -a -G root test_user```
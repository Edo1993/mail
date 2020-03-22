# 30_mail
# Почта: SMTP, IMAP, POP3 

# Домашнее задание
установка почтового сервера
1. Установить в виртуалке postfix+dovecot для приёма почты на виртуальный домен любым обсужденным на семинаре способом
2. Отправить почту телнетом с хоста на виртуалку
3. Принять почту на хост почтовым клиентом

Результат
1. Полученное письмо со всеми заголовками
2. Конфиги postfix и dovecot

__________________________________________________________________________________________________________________________

# Итог:

Очень помогла [статья](https://itdoxy.com/%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D0%BB%D0%BE%D0%BA%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BF%D0%BE%D1%87%D1%82%D0%BE%D0%B2%D0%BE%D0%B3%D0%BE-%D1%81%D0%B5%D1%80%D0%B2/), собственно, всё по ней и сделано.

[Vagrantfile](https://github.com/Edo1993/mail/blob/master/Vagrantfile) + [конфиги](https://github.com/Edo1993/mail/tree/master/configs) 

Отправки письма через telnet

```
ed@RobotUnicorn:~/otus/post/$ telnet 192.168.11.150 25
Trying 192.168.11.150...
Connected to 192.168.11.150.
Escape character is '^]'.
220 server1.unixmen.local ESMTP Postfix
EHLO server1.unixmen.local
250-server1.unixmen.local
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
mail from: <rd4th@mail.ru>
250 2.1.0 Ok
rcpt to:<sk@unixmen.local>
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
From: Karina <rd4th@mail.ru>
To: Karina <sk@unixmen.local>
Subject: Test
Content-Type: text/plain

PLease, work
.
250 2.0.0 Ok: queued as 1CD3260EC
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

Получение
![Img_alt](https://github.com/Edo1993/mail/blob/master/301.png)

__________________________________________________________________________________________________________________________

# Ход работы

Разворачиваем vm через ```vagrant up```, подключаемся ```vagrant ssh```.

Отключить SELINUX ```setenforce 0```.

Установить репозиторий EPEL ```yum install epel-release -y```

1) Установка Postfix

```
yum install postfix -y
```
Отредактируйте файл [/etc/postfix/main.cf](https://github.com/Edo1993/mail/blob/master/configs/etc/postfix/main.cf)

```
vi /etc/postfix/main.cf
```
Редактировались параметры:

mynetworks из статьи - не трогать
```
myhostname = server1.unixmen.local
mydomain = unixmen.local
inet_interfaces = all
inet_protocols = all
#mydestination = $myhostname, localhost.$mydomain, localhost
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
home_mailbox = Maildir/
```

Запустите/перезапустите (enable/restart) сервис Postfix

```
systemctl enable postfix
systemctl restart postfix
```

Создать тестового пользователя с именем “SK” и задать пароль

```
useradd sk
passwd sk
```

2) Установка Dovecot

Dovecot — это почтовый IMAP и POP3-сервер (с открытым исходным кодом) для Unix/Linux.

```
yum install dovecot -y
```

Настройка Dovecot.

- Отредактируйте файл [/etc/dovecot/dovecot.conf](https://github.com/Edo1993/mail/blob/master/configs/etc/dovecot/dovecot.conf),

```
vi /etc/dovecot/dovecot.conf
```

Раскомментируйте следующую строку - Line 24:

```
protocols = imap pop3 lmtp
```

- Отредактируйте файл [/etc/dovecot/conf.d/10-mail.conf](https://github.com/Edo1993/mail/blob/master/configs/etc/dovecot/conf.d/10-mail.conf),

```
vi /etc/dovecot/conf.d/10-mail.conf
```

Внесите следующие изменения - Line 24:

```
mail_location = maildir:~/Maildir
```

- Отредактируйте файл [/etc/dovecot/conf.d/10-auth.conf](https://github.com/Edo1993/mail/blob/master/configs/etc/dovecot/conf.d/10-auth.conf)

```
vi /etc/dovecot/conf.d/10-auth.conf
```

Внесите следующие изменения:

```
## Line 10 - раскомментируйте ##
disable_plaintext_auth = yes

## Line 100 - Допишите слово: "login" ##
auth_mechanisms = plain login
```

- Отредактируйте файл [/etc/dovecot/conf.d/10-master.conf](https://github.com/Edo1993/mail/edit/master/configs/etc/dovecot/conf.d/10-master.conf),

```
vi /etc/dovecot/conf.d/10-master.conf
```

Внесите следующие изменения:

```
## Line 91, 92 - Раскомментируйте строки и добавьте "postfix" ##
#mode = 0600
 user = postfix
 group = postfix
[...]
```

Запустить сервис Dovecot:

```
systemctl enable dovecot
systemctl start dovecot
```

3) Установка Squirrelmail

```
yum install squirrelmail -y
```

Настройка Squirrelamil. Перейдите в директорию /usr/share/squirrelmail/config/:

```
cd /usr/share/squirrelmail/config/
```

Запустите следующую команду для настройки Squirrelmail:

```
./conf.pl
```

Откроется соответствующий мастер установки. Введите “1”, чтобы настроить сведения об организации. Откроется соответствующий мастер установки. Снова введите “1”, чтобы изменить сведения об организации. Введите желаемое имя для вашей организации, после чего нажмите Enter. Аналогично, настройте все оставшиеся пункты (описание, логотип, имя провайдера и т.д.). Когда закончите настройку, нажмите “S”, чтобы сохранить изменения, и “R”, чтобы вернуться обратно в главное меню.
Настраивалось по статье, поэтому в итоге моя конфигурация выглядит так:

```
SquirrelMail Configuration : Read: config.php
Config version 1.4.0; SquirrelMail version unknown
---------------------------------------------------------
Organization Preferences
1.  Organization Name      : Unixmen
2.  Organization Logo      : ../images/sm_logo.png
3.  Org. Logo Width/Height : (308/111)
4.  Organization Title     : SquirrelMail $version
5.  Signout Page           : 
6.  Top Frame              : _top
7.  Provider link          : http://squirrelmail.org/
8.  Provider name          : Unixmen Mail

R   Return to Main Menu
C   Turn color off
S   Save data
Q   Quit
```

Теперь, введите “2” для настройки параметров почтового сервера (таких как имя домена, почтовый агент и т.д.).
Введите “1”, затем введите желаемый домен для почты (например unixmen.local) и нажмите Enter. 
Введите “3” и замените агента пересылки сообщений (MTA) sendmail на Postfix (то есть SMTP).
Введите “2”, чтобы переключиться с sendmail на Postfix.

*конфигурация в данном разделе*

```
SquirrelMail Configuration : Read: config.php
Config version 1.4.0; SquirrelMail version unknown
---------------------------------------------------------
Server Settings

General
-------
1.  Domain                 : unixmen.local
2.  Invert Time            : false
3.  Sendmail or SMTP       : SMTP

A.  Update IMAP Settings   : localhost:143 (uw)
B.  Update SMTP Settings   : localhost:25

R   Return to Main Menu
C   Turn color off
S   Save data
Q   Quit
```

Теперь введите “S” и следом “Q”, чтобы сохранить настройки Squirrelmail и выйти.

4) Создать виртуальный хост (vhost) Squirrelmail в файле конфигурации Apache.

```
vi /etc/httpd/conf/httpd.conf
```

Добавить в конец следующие строчки:

```
Alias /webmail /usr/share/squirrelmail
<Directory /usr/share/squirrelmail>
 Options Indexes FollowSymLinks 
 RewriteEngine On
 AllowOverride All
 DirectoryIndex index.php
 Order allow,deny
 Allow from all
</Directory>
```

Перезагрузить сервис Apache:

```
systemctl restart httpd
```

Доступ к Webmail

```
http://192.168.11.150/webmail/
```

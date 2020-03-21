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

Итог:

Очень помогла [статья](https://itdoxy.com/%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D0%BB%D0%BE%D0%BA%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BF%D0%BE%D1%87%D1%82%D0%BE%D0%B2%D0%BE%D0%B3%D0%BE-%D1%81%D0%B5%D1%80%D0%B2/), собственно, всё по ней и сделано.

[Vagrantfile](https://github.com/Edo1993/mail/blob/master/Vagrantfile) + [конфиги](https://github.com/Edo1993/mail/tree/master/configs) 

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

2) Тестирование Postfix почтового сервера

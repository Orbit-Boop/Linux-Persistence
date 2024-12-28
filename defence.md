# Для защиты

## Копирование файлов
На резервный диск скопировать passwd, sudoers, shadow, group. После атаки восстанавливаем эти файлы.

## White list
ip и порты

## demon с ssh-ключом
```
useradd -M -u <UID> -g <GID> -G <addition groups> <username>
```
sshd_config
```
Match group <groupname>
AuthorizedKeysFile /etc/ssh/authorized-keys/%u
```
restart ssh

## port knocking

## Проверить процессы
```
ps auxf
```

## Проверить cron
```
crontab -e
```
/etc/cron.d/

## Найти недавно измененные файлы
find /home/captain -type f -mmin -30

## Найти файлы по типу rc.local

## Firewall
Защита от bind-shell и подключения по ssh
```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow from <IP address>
sudo ufw allow from <IP address> to any port <PORT>
```

## Логи
### Настройка сервера

Открываем конфигурационный файл:
```
vi /etc/rsyslog.conf
```
Снимаем комментарии со следующих строк:
```
$ModLoad imudp
$UDPServerRun 514

$ModLoad imtcp
$InputTCPServerRun 514
```
* в данном примере мы разрешили запуск сервера для соединений TCP и UDP на портах 514. На самом деле, можно оставить только один протокол, например, более безопасный и медленный TCP.

После добавляем в конфигурационный файл строки:
```
$template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& ~
```
* в данном примере мы создаем шаблон с названием RemoteLogs, который принимает логи всех категорий, любого уровня (про категории и уровни читайте ниже); логи, полученный по данному шаблону будут сохраняться в каталоге по маске /var/log/rsyslog/<имя компьютера, откуда пришел лог>/<приложение, чей лог пришел>.log; конструкция & ~ говорит о том, что после получения лога, необходимо остановить дальнейшую его обработку.

Перезапускаем службу логов:
```
systemctl restart rsyslog
```

### Настройка клиента

rsyslog.conf
```
###############
#### RULES ####
###############

*.*;auth,authpriv.none		-/media/data/syslog

auth,authpriv.*			/media/data/auth.log
cron.*				-/media/data/cron.log
kern.*				-/media/data/kern.log
mail.*				-/media/data/mail.log
news.*				-/media/data/news.log
user.*				-/media/data/user.log
security.*			-/media/data/sec.log

#
# Emergencies are sent to everybody logged in.
#
*.emerg				:omusrmsg:*

```

Все логи

Для начала можно настроить отправку всех логов на сервер. Создаем конфигурационный файл для rsyslog:
```
vi /etc/rsyslog.d/all.conf
```
Добавляем:
```
*.* @@192.168.0.15:514
```
* где 192.168.0.15 —  IP-адрес сервера логов. *.* — перенаправлять любой лог.

Перезапускаем rsyslog:
```
systemctl restart rsyslog
```
Для определенных категорий

Если необходимо отправлять только определенные категории логов, создаем конфигурационный файл для соответствующей, например:
```
vi /etc/rsyslog.d/kern.conf
```
Добавим строку:
```
kern.* @@192.168.0.15:514
```
Перезапускаем сервис логов:
```
systemctl restart rsyslog
```
Возможные категории для логов (facility):
№ 	Категория 	Описание
0 	kern 	Сообщения, отправляемые ядром
1 	user 	Пользовательские программы
2 	mail 	Почта
3 	daemon 	Сервисы (демоны)
4 	auth 	Безопасность/вход в систему/аутентификация
5 	syslog 	Сообщения от syslog
6 	lpr 	Логи печати
7 	news 	Новостные группы (usenet)
8 	uucp 	Unix-to-Unix CoPy (копирование файлов между компьютерами)
9 	cron 	Планировщик заданий
10 	authpriv 	Безопасность/вход в систему/аутентификация - защищенный режим
11 	ftp 	Логи при передачи данных по FTP
12 	ntp 	Лог службы синхронизации времени (существует не везде)
13 	security, log audit 	Журнал аудита (существует не везде)
14 	console, log alert 	Сообщения, отправляемые в консоль (существует не везде)
15 	solaris-cron, clock daemon 	Cron в solaris (существует не везде)
16-23 	local0 - local7 	Зарезервированы для локального использования. Уровень серьезности определяется числом от 0 до 7.
Для определенного уровня

Если мы хотим передавать только сообщения об ошибках, добавляем строку в файл конфигурации rsyslog:
```
vi /etc/rsyslog.d/erors.conf

*.err @@192.168.0.15:514
```
Перезапускаем сервис логов:
```
systemctl restart rsyslog
```
Возможные уровни логов:

Возможные категории для логов (severity):
№ 	Уровень 	Расшифровка
0 	emerg 	Система не работает (PANIC)
1 	alert 	Серьезная проблема, требующая внимания
2 	crit 	Критическая ошибка
3 	err 	Ошибка (ERROR)
4 	warning 	Предупреждение (WARN)
5 	notice 	Важное информационное сообщение
6 	info 	Информационное сообщение
7 	debug 	Отладочная информация
Аудит определенного лог-файла

Мы можем настроить слежение за изменением определенного лога и передавать их на сервер. Для этого нужно настроить и сервер, и клиента.
Настройка клиента

Создаем новый конфигурационный файл:
```
vi /etc/rsyslog.d/audit.conf

$ModLoad imfile
$InputFileName /var/log/audit/audit.log
$InputFileTag tag_audit_log:
$InputFileStateFile audit_log
$InputFileSeverity info
$InputFileFacility local6
$InputRunFileMonitor

*.*   @@192.168.0.15:514
```
* в данном примере мы будем отслеживать изменения лог-файла /var/log/audit/audit.log; нас интересуют события от уровня info и выше; все события будет отмечены категорией local6 и переданы на сервер 192.168.0.15.

Перезапускаем сервис на клиенте:
```
systemctl restart rsyslog
```

## Аудит
```
sudo auditctl -w /etc/passwd -p wa -k passwd_checker

sudo auditctl -w /etc/ -p wa -k etc_checker
sudo auditctl -a always,exit -F dir=/etc/ -F perm=wa -k etc_checker

sudo auditctl -w /home/ -p wa -k home_checker
sudo auditctl -w /root/ -p wa -k root_checker
```

audit.rules
```
## This file is automatically generated from /etc/audit/rules.d
-D
-b 8192
-f 1
--backlog_wait_time 60000

# отслеживать системные вызовы unlink () и rmdir()
-a exit,always -S unlink -S rmdir

# отслеживать доступ к файлам паролей и групп и попытки их изменения:
-w /etc/group -p wa -k g_au
-w /etc/passwd -p wa -k p_au
-w /etc/shadow -p wa -k sh_au
-w /etc/sudoers -p wa -k su_au
-a always,exit -F dir=/root/ -F perm=wa -k r_au
-a always,exit -F dir=/home/ -F perm=wa -k h_au

# закрыть доступ к конфигурационному файлу для предотвращения изменений
-e 2
```

## Перезапуск служб аудита и логгирования
```
crontab -e
* * * * * /.../start.sh
```

start.sh
```
systemctl enable --now auditd
systemctl enable --now rsyslog
systemctl enable --now systemd-journald
```

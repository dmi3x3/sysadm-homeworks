# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

Ответ:
Установка производится запуском однострочника, результатом его работы является запущенный сервис node_exporter.service

```bash
vagrant@vagrant:~/node_exp$ cd /opt && sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz \ 
&& sudo tar xzf node_exporter-1.3.1.linux-amd64.tar.gz && sudo rm -f node_exporter-1.3.1.linux-amd64.tar.gz \
&& echo "EXTRA_OPTS=\"--log.level=info\"" | sudo tee /etc/node_exporter.conf \
&& sudo mkdir -p /usr/local/lib/systemd/system/ \
&& echo -e "# /usr/local/lib/systemd/system/node_exporter.service \n[Unit] \nDescription=\"node_exporer service file for home works netology 03-sysadmin-04-os\"\n\n[Service]\nEnvironmentFile=/etc/node_exporter.conf\nExecStart=/opt/node_exporter-1.3.1.linux-amd64/node_exporter \$EXTRA_OPTS\nIgnoreSIGPIPE=false\nKillMode=process\nRestart=on-failure\nStandardOutput=file:/var/log/node_explorer.log\nStandardError=file:/var/log/node_explorer.log\n\n[Install]\nWantedBy=multi-user.target" \
| sudo tee /usr/local/lib/systemd/system/node_exporter.service && sudo systemctl daemon-reload \
&& sudo systemctl enable node_exporter.service && sudo systemctl start node_exporter.service

--2022-01-05 04:53:44--  https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.121.4
Connecting to github.com (github.com)|140.82.121.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/7c60f6f9-7b41-446c-be81-a6c24a9d0383?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220105%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220105T045344Z&X-Amz-Expires=300&X-Amz-Signature=c907623ae8c83487bde2b6bfc115b05f21ae8d9f0dfdf2227924ef95370c1a69&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.3.1.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-01-05 04:53:44--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/7c60f6f9-7b41-446c-be81-a6c24a9d0383?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220105%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220105T045344Z&X-Amz-Expires=300&X-Amz-Signature=c907623ae8c83487bde2b6bfc115b05f21ae8d9f0dfdf2227924ef95370c1a69&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.3.1.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.110.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9033415 (8.6M) [application/octet-stream]
Saving to: ‘node_exporter-1.3.1.linux-amd64.tar.gz’

node_exporter-1.3.1.linux-amd64.tar.gz             100%[================================================================================================================>]   8.61M  8.44MB/s    in 1.0s    

2022-01-05 04:53:46 (8.44 MB/s) - ‘node_exporter-1.3.1.linux-amd64.tar.gz’ saved [9033415/9033415]

EXTRA_OPTS="--log.level=info"
# /usr/local/lib/systemd/system/node_exporter.service 
[Unit] 
Description="node_exporer service file for home works netology 03-sysadmin-04-os"

[Service]
EnvironmentFile=/etc/node_exporter.conf
ExecStart=/opt/node_exporter-1.3.1.linux-amd64/node_exporter $EXTRA_OPTS
IgnoreSIGPIPE=false
KillMode=process
Restart=on-failure
StandardOutput=file:/var/log/node_explorer.log
StandardError=file:/var/log/node_explorer.log

[Install]
WantedBy=multi-user.target
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /usr/local/lib/systemd/system/node_exporter.service.
vagrant@vagrant:/opt$ systemctl status node_exporter
● node_exporter.service - "node_exporer service file for home works netology 03-sysadmin-04-os"
     Loaded: loaded (/usr/local/lib/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-01-05 04:53:47 UTC; 36s ago
   Main PID: 58490 (node_exporter)
      Tasks: 6 (limit: 1304)
     Memory: 2.6M
     CGroup: /system.slice/node_exporter.service
             └─58490 /opt/node_exporter-1.3.1.linux-amd64/node_exporter --log.level=info

Jan 05 04:53:47 vagrant systemd[1]: Started "node_exporer service file for home works netology 03-sysadmin-04-os".
```

```bash
vagrant@vagrant:/opt$ sudo systemctl stop node_exporter
vagrant@vagrant:/opt$ systemctl status node_exporter
● node_exporter.service - "node_exporer service file for home works netology 03-sysadmin-04-os"
     Loaded: loaded (/usr/local/lib/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Wed 2022-01-05 04:56:03 UTC; 1s ago
    Process: 58490 ExecStart=/opt/node_exporter-1.3.1.linux-amd64/node_exporter $EXTRA_OPTS (code=killed, signal=TERM)
   Main PID: 58490 (code=killed, signal=TERM)

Jan 05 04:53:47 vagrant systemd[1]: Started "node_exporer service file for home works netology 03-sysadmin-04-os".
Jan 05 04:56:03 vagrant systemd[1]: Stopping "node_exporer service file for home works netology 03-sysadmin-04-os"...
Jan 05 04:56:03 vagrant systemd[1]: node_exporter.service: Succeeded.
Jan 05 04:56:03 vagrant systemd[1]: Stopped "node_exporer service file for home works netology 03-sysadmin-04-os".
vagrant@vagrant:/opt$ sudo systemctl start node_exporter
vagrant@vagrant:/opt$ systemctl status node_exporter
● node_exporter.service - "node_exporer service file for home works netology 03-sysadmin-04-os"
     Loaded: loaded (/usr/local/lib/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-01-05 04:56:12 UTC; 1s ago
   Main PID: 58520 (node_exporter)
      Tasks: 5 (limit: 1304)
     Memory: 2.7M
     CGroup: /system.slice/node_exporter.service
             └─58520 /opt/node_exporter-1.3.1.linux-amd64/node_exporter --log.level=info

Jan 05 04:56:12 vagrant systemd[1]: Started "node_exporer service file for home works netology 03-sysadmin-04-os".
vagrant@vagrant:/opt$ sudo systemctl restart node_exporter
vagrant@vagrant:/opt$ systemctl status node_exporter
● node_exporter.service - "node_exporer service file for home works netology 03-sysadmin-04-os"
     Loaded: loaded (/usr/local/lib/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-01-05 04:56:28 UTC; 2s ago
   Main PID: 58546 (node_exporter)
      Tasks: 5 (limit: 1304)
     Memory: 2.5M
     CGroup: /system.slice/node_exporter.service
             └─58546 /opt/node_exporter-1.3.1.linux-amd64/node_exporter --log.level=info

Jan 05 04:56:28 vagrant systemd[1]: Started "node_exporer service file for home works netology 03-sysadmin-04-os".
```

2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.

Ответ: 
```bash
node_cpu_seconds_total{cpu="0",mode="idle"} 15766.17
node_cpu_seconds_total{cpu="0",mode="iowait"} 0.64
node_cpu_seconds_total{cpu="0",mode="softirq"} 1.22
node_cpu_seconds_total{cpu="0",mode="steal"} 0
node_cpu_seconds_total{cpu="0",mode="system"} 12.76
node_cpu_seconds_total{cpu="0",mode="user"} 3.45
node_cpu_seconds_total{cpu="1",mode="idle"} 15765.25
node_cpu_seconds_total{cpu="1",mode="iowait"} 1.16
node_cpu_seconds_total{cpu="1",mode="softirq"} 4.48
node_cpu_seconds_total{cpu="1",mode="steal"} 0
node_cpu_seconds_total{cpu="1",mode="system"} 11.02
node_cpu_seconds_total{cpu="1",mode="user"} 4.03
node_cpu_seconds_total{cpu="2",mode="idle"} 15789.65
node_cpu_seconds_total{cpu="2",mode="iowait"} 1.44
node_cpu_seconds_total{cpu="2",mode="softirq"} 0.5
node_cpu_seconds_total{cpu="2",mode="steal"} 0
node_cpu_seconds_total{cpu="2",mode="system"} 11.4
node_cpu_seconds_total{cpu="2",mode="user"} 3.45

node_memory_MemAvailable_bytes 8.85825536e+08
node_memory_MemFree_bytes 6.61721088e+08
node_memory_MemTotal_bytes 1.232416768e+09
node_memory_SwapCached_bytes 0
node_memory_SwapFree_bytes 1.027600384e+09
node_memory_SwapTotal_bytes 1.027600384e+09

node_disk_io_time_seconds_total{device="dm-0"} 8.132
node_disk_io_time_seconds_total{device="dm-1"} 0.04
node_disk_io_time_seconds_total{device="sda"} 8.22
node_disk_io_time_weighted_seconds_total{device="dm-0"} 12.932
node_disk_io_time_weighted_seconds_total{device="dm-1"} 0.02
node_disk_io_time_weighted_seconds_total{device="sda"} 1.8760000000000001
node_disk_read_bytes_total{device="dm-0"} 2.95699456e+08
node_disk_read_bytes_total{device="dm-1"} 3.342336e+06
node_disk_read_bytes_total{device="sda"} 3.09482496e+08
node_disk_read_time_seconds_total{device="dm-0"} 6.496
node_disk_read_time_seconds_total{device="dm-1"} 0.02
node_disk_read_time_seconds_total{device="sda"} 2.614
node_disk_reads_completed_total{device="dm-0"} 8893
node_disk_reads_completed_total{device="dm-1"} 146
node_disk_reads_completed_total{device="sda"} 5907
node_disk_write_time_seconds_total{device="dm-0"} 6.436
node_disk_write_time_seconds_total{device="dm-1"} 0
node_disk_write_time_seconds_total{device="sda"} 5.3660000000000005
node_disk_writes_completed_total{device="dm-0"} 7351
node_disk_writes_completed_total{device="dm-1"} 0
node_disk_writes_completed_total{device="sda"} 5121
node_disk_written_bytes_total{device="dm-0"} 1.08679168e+08
node_disk_written_bytes_total{device="dm-1"} 0
node_disk_written_bytes_total{device="sda"} 1.07574272e+08

node_filesystem_avail_bytes{device="/dev/mapper/vgvagrant-root",fstype="ext4",mountpoint="/"} 6.0626763776e+10
node_filesystem_avail_bytes{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"} 5.35801856e+08
node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 1.22560512e+08
node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run/lock"} 5.24288e+06
node_filesystem_avail_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run/user/1000"} 1.23240448e+08
node_filesystem_avail_bytes{device="vagrant",fstype="vboxsf",mountpoint="/vagrant"} 2.58685394944e+11
node_filesystem_readonly{device="/dev/mapper/vgvagrant-root",fstype="ext4",mountpoint="/"} 0
node_filesystem_readonly{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"} 0
node_filesystem_readonly{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 0
node_filesystem_readonly{device="tmpfs",fstype="tmpfs",mountpoint="/run/lock"} 0
node_filesystem_readonly{device="tmpfs",fstype="tmpfs",mountpoint="/run/user/1000"} 0
node_filesystem_readonly{device="vagrant",fstype="vboxsf",mountpoint="/vagrant"} 0
node_filesystem_size_bytes{device="/dev/mapper/vgvagrant-root",fstype="ext4",mountpoint="/"} 6.5827115008e+10
node_filesystem_size_bytes{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"} 5.35805952e+08
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run"} 1.23244544e+08
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run/lock"} 5.24288e+06
node_filesystem_size_bytes{device="tmpfs",fstype="tmpfs",mountpoint="/run/user/1000"} 1.23240448e+08
node_filesystem_size_bytes{device="vagrant",fstype="vboxsf",mountpoint="/vagrant"} 4.96938336256e+11

node_network_carrier_changes_total{device="eth0"} 2
node_network_carrier_down_changes_total{device="eth0"} 1
node_network_carrier_up_changes_total{device="eth0"} 1
node_network_info{address="08:00:27:73:60:cf",broadcast="ff:ff:ff:ff:ff:ff",device="eth0",duplex="full",ifalias="",operstate="up"} 1
node_network_mtu_bytes{device="eth0"} 1500
node_network_receive_bytes_total{device="eth0"} 2.1106462e+07
node_network_receive_drop_total{device="eth0"} 0
node_network_receive_errs_total{device="eth0"} 0
node_network_receive_multicast_total{device="eth0"} 0
node_network_receive_packets_total{device="eth0"} 33958
node_network_speed_bytes{device="eth0"} 1.25e+08
node_network_transmit_bytes_total{device="eth0"} 1.761534e+06
node_network_transmit_drop_total{device="eth0"} 0
node_network_transmit_errs_total{device="eth0"} 0
node_network_transmit_packets_total{device="eth0"} 18512
node_network_transmit_queue_length{device="eth0"} 1000
node_network_up{device="eth0"} 1
```

3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

Ответ:

![img.png](img.png)

4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Ответ: Да
```bash
vagrant@vagrant:~$ dmesg | grep virt
[    0.001846] CPU MTRRs all blank - virtualized system.
[    0.096151] Booting paravirtualized kernel on KVM
[    2.868735] systemd[1]: Detected virtualization oracle.
```
В первой строчке видно, что ОС осознала свою виртуализированность. А в третьей строке видно вендора системы виртуализации.

5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?

Ответ:

    В man proc про параметр /proc/sys/fs/nr_open написано:
        Этот файл установливает верхний предел на значение, 
        до которого может быть увеличен предел ресурса RLIMIT_NOFILE 
        (см. getrlimit (2)). Этот потолок применяется как для непривилегированного, 
        так и для привилегированного процесса. Значение по умолчанию 
        в этом файле - 1048576. (До Linux 2.6.25 потолок для RLIMIT_NOFILE был 
        жестко закодирован до того же значения.)

    В man по системному вызову getrlimit написано:
           RLIMIT_NOFILE
              Это указывает значение на единицу больше максимального файла
              номер дескриптора, который может быть открыт этим процессом.
              Попытки ( open (2) , pipe (2) , dup (2) и т. Д.) Превысить это
              limit выдает ошибку EMFILE . (Исторически этот предел
              был назван RLIMIT_OFILE в BSD.)

              Начиная с Linux 4.5 этот предел также определяет максимальное
              количество файловых дескрипторов, которые непривилегированный процесс
              (один без возможности CAP_SYS_RESOURCE ) может иметь "в
              полет »к другим процессам, передаваясь через UNIX
              доменные сокеты. Это ограничение применяется к 
              системному вызову sendmsg (2) . Для получения дополнительной информации см. Unix (7) .

Проще говоря в fs.nr_open записано значение максимального количества открытых файлов в одном процессе.
в тестовой системе установлено значение по-умолчанию.
```bash
vagrant@vagrant:~$ sudo sysctl fs.nr_open
fs.nr_open = 1048576
```
Не позволит достичь такого числа ограничение количества открытых файлов на пользователя:

```bash
vagrant@vagrant:~$ ulimit -Sn
1024
```
Оно может быть изменено, в пределах сессии

```bash
vagrant@vagrant:~$ ulimit -Sn 4096
vagrant@vagrant:~$ ulimit -Sn
4096
```
Или постоянно в файле /etc/security/limits.conf или файле *.conf директории /etc/security/limits.d

```bash
vagrant@vagrant:~$ cat /etc/security/limits.d/user.conf 
vagrant            hard    nofile            1000000
vagrant            soft    nofile            4096
```
После изменения файла требуется перелогиниться.

6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.

Ответ:
```bash
vagrant@vagrant:~$ screen -S sleep_1h
vagrant@vagrant:~$ sudo -i
root@vagrant:~# unshare -f --pid --mount-proc sleep 1h
ctrl a d
root@vagrant:~# ps aux | grep "sleep 1h"
root        1502  0.0  0.0   8080   580 pts/1    T    02:34   0:00 unshare -f --pid --mount-proc sleep 1h
root        1503  0.0  0.0   8076   528 pts/1    S    02:34   0:00 sleep 1h
root        1506  0.0  0.0   9032   740 pts/1    S+   02:34   0:00 grep --color=auto sleep 1h
vagrant@vagrant:~$ screen -S sudo_session
vagrant@vagrant:~$ sudo -i
root@vagrant:~# nsenter --target 1503 --pid --mount
root@vagrant:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   8076   528 pts/1    S    02:34   0:00 sleep 1h
root           2  0.1  0.3   9836  4016 pts/1    S    02:35   0:00 -bash
root          11  0.0  0.2  11492  3384 pts/1    R+   02:35   0:00 ps aux
```

7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Ответ:

:(){ :|:& };: - Логическая бомба (известная также как fork bomb), забивающая память системы, что в итоге приводит к её зависанию.
Для наглядности немного преобразуем ее:
```shell
fu() {
fu | fu &
}
fu
```
Этот Bash код создаёт функцию, которая запускает ещё два своих экземпляра, которые, в свою очередь снова запускают эту функцию и так до тех пор, пока этот процесс не займёт всю физическую память компьютера, и он просто не зависнет.

Из вывода dmesg видно, что сработало ограничение контрольной группы (cgroup),  на количество процессов пользователя.
```bash
[Wed Jan  5 02:49:44 2022] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-7.scope
```
Статус пользователя во время работы "Вилочной бомбы" выглядит примерно так, в строке "Tasks: 2846 (limit: 2869)" укакзано число запущенных процессов и лимит.
```bash
● user-1000.slice - User Slice of UID 1000
     Loaded: loaded
    Drop-In: /usr/lib/systemd/system/user-.slice.d
             └─10-defaults.conf
     Active: active since Wed 2022-01-05 00:39:17 UTC; 2h 58min ago
       Docs: man:user@.service(5)
      Tasks: 2846 (limit: 2869)
     Memory: 712.0M
     CGroup: /user.slice/user-1000.slice
             ├─session-7.scope
             │ ├─ 1425 sshd: vagrant [priv]
             │ ├─ 1455 sshd: vagrant@pts/0
             │ ├─ 1456 -bash
             │ ├─ 1471 SCREEN
             │ ├─ 1472 /bin/bash
             │ ├─ 1545 screen -x
             │ ├─ 1547 sudo -i
             │ ├─ 1548 -bash
             │ ├─ 1567 SCREEN -S ruser
             │ ├─ 1568 /bin/bash
             │ ├─38736 /bin/bash
             │ ├─38754 /bin/bash
             │ ├─38761 /bin/bash
             │ ├─38764 /bin/bash
             │ ├─38801 /bin/bash
             │ ├─38932 /bin/bash
             │ ├─38933 /bin/bash
             │ ├─38938 /bin/bash
             │ ├─38949 /bin/bash
             │ ├─38999 /bin/bash
             │ ├─39061 /bin/bash
             │ ├─39077 /bin/bash
             │ ├─39078 /bin/bash
             │ ├─39084 /bin/bash
```


Один из вариантов изменения общего лимита для пользователя с uid 1000 - создать файл /etc/systemd/system/user-1000.slice.d/override.conf со следующим содержимым:
```text
[Slice] 
TasksMax = 2500
```
Здесь указывется число процессов, в данном случае 2500, для отключения лимита надо указать - infinity.
```bash
vagrant@vagrant:~$ sudo mkdir /etc/systemd/system/user-1000.slice.d
vagrant@vagrant:~$ echo -e "[Slice] \nTasksMax = 2500" | sudo tee /etc/systemd/system/user-1000.slice.d/override1.conf
vagrant@vagrant:~$ sudo systemctl daemon-reload
vagrant@vagrant:~$ systemctl status user-1000.slice
● user-1000.slice - User Slice of UID 1000
     Loaded: loaded
    Drop-In: /usr/lib/systemd/system/user-.slice.d
             └─10-defaults.conf
             /etc/systemd/system/user-1000.slice.d
             └─override.conf
     Active: active since Wed 2022-01-05 00:39:17 UTC; 3h 16min ago
       Docs: man:user@.service(5)
      Tasks: 14 (limit: 2500)
     Memory: 46.6M
     CGroup: /user.slice/user-1000.slice
             ├─session-7.scope
             │ ├─ 1425 sshd: vagrant [priv]
             │ ├─ 1455 sshd: vagrant@pts/0
             │ ├─ 1456 -bash
             │ ├─ 1471 SCREEN
             │ ├─ 1472 /bin/bash
             │ ├─ 1545 screen -x
             │ ├─ 1547 sudo -i
             │ ├─ 1548 -bash
             │ ├─ 1567 SCREEN -S ruser
             │ ├─ 1568 /bin/bash
             │ ├─58355 systemctl status user-1000.slice
             │ └─58356 pager
             └─user@1000.service
               └─init.scope
                 ├─842 /lib/systemd/systemd --user
                 └─843 (sd-pam)
```

Tasks: 14 (limit: 2500)
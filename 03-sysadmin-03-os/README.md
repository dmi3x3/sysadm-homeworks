# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, поэтому запустить strace непосредственно на cd не получится. Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. Вам нужно найти тот единственный, который относится именно к cd.

Ответ:
```bash
vagrant@vagrant:~$ strace /bin/bash -c 'cd /tmp' 2>&1 | grep "\/tmp"
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffdf93836e0 /* 24 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```

cd делает системный вызов - chdir("/tmp") 


2. Попробуйте использовать команду file на объекты разных типов на файловой системе. Например:
```bash
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
```
Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.

Ответ:
База данных - файл с магическими шаблонами для команды "file" (file command's magic pattern file)
```shell
vagrant@vagrant:~$ ls -lh /usr/lib/file/magic.mgc
-rw-r--r-- 1 root root 5.6M Jan 16  2020 /usr/lib/file/magic.mgc
```

```shell
vagrant@vagrant:~$ strace file /dev/tty 2>&1 | grep magic
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
stat("/home/vagrant/.magic.mgc", 0x7fff9a4c7120) = -1 ENOENT (No such file or directory)
stat("/home/vagrant/.magic", 0x7fff9a4c7120) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
```

в выводе strace видно символическую ссылку на этот файл.
```shell
vagrant@vagrant:~$ ls -lh /usr/share/misc/magic.mgc
lrwxrwxrwx 1 root root 24 Jul 28 17:46 /usr/share/misc/magic.mgc -> ../../lib/file/magic.mgc
```
выводе strace видно, что 'file' ищет свою БД так же и в фалах пользователя.
кроме того, идет поиск конфигурационного файла команды file (/etc/magic), который содержит описания различных форматов файлов, опираясь на которые эта команда определяет тип файла. Еще мы видим библиотеку для работы с файлом magic.mgc - тоже симлинк: /lib/x86_64-linux-gnu/libmagic.so.1 -> libmagic.so.1.0.0


3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

Ответ:
    Запустим сессию в screen, а в ней текстовый редактор vi  и напишем/сохраним, что-нибудь в файле, редактор оставим открытым. Отключимся от сессии. Узнаем PID запущенного vi по имени файла, открытого в нем. Удаляем файл /home/vagrant/.file_test.txt.swp Программой lsof номеру PID находим файл со статусом "deleted". Объем файла 12288 байт, его дескриптор 5, очищаем файл. Смотрим размер файла - он нулевой.  
```bash
vagrant@vagrant:~$ screen -S vi_test_file
vagrant@vagrant:~$ vi /home/vagrant/file_test.txt
[detached from 1102.vi_test_file]
vagrant@vagrant:~$ ps aux | grep file_test
vagrant     1854  0.0  0.8  24536  9708 pts/1    S+   07:08   0:00 vi file_test.txt
vagrant     1859  0.0  0.0   9032   736 pts/0    S+   07:09   0:00 grep --color=auto file_test
vagrant@vagrant:~$ lsof -p 1854 | grep deleted
vi      1854 vagrant    5u   REG  253,0    12288  131084 /home/vagrant/.file_test.txt.swp (deleted)
vagrant@vagrant:~$ >/proc/1854/fd/5 
vagrant@vagrant:~$ lsof -p 1854 | grep deleted
vi      1854 vagrant    5u   REG  253,0        0  131084 /home/vagrant/.file_test.txt.swp (deleted)
```
обнулить файл можно командами: >/proc/1854/fd/5 или echo -n "" > /proc/1854/fd/5

4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Ответ:
Зомби не занимают памяти (как процессы-сироты), но блокируют записи в таблице процессов, размер которой ограничен для каждого пользователя и системы в целом.

При достижении лимита записей все процессы пользователя, от имени которого выполняется создающий зомби родительский процесс, не будут способны создавать новые дочерние процессы. Кроме этого, пользователь, от имени которого выполняется родительский процесс, не сможет зайти на консоль (локальную или удалённую) или выполнить какие-либо команды на уже открытой консоли (потому что для этого командный интерпретатор sh должен создать новый процесс), и для восстановления работоспособности (завершения виновной программы) будет необходимо вмешательство системного администратора.


5. В iovisor BCC есть утилита opensnoop:

```bash
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
```

На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools для Ubuntu 20.04. Дополнительные сведения по установке.

Ответ:
```bash
vagrant@vagrant:~$ sudo /usr/sbin/opensnoop-bpfcc 
PID    COMM               FD ERR PATH
617    irqbalance          6   0 /proc/interrupts
617    irqbalance          6   0 /proc/stat
617    irqbalance          6   0 /proc/irq/20/smp_affinity
617    irqbalance          6   0 /proc/irq/0/smp_affinity
617    irqbalance          6   0 /proc/irq/1/smp_affinity
617    irqbalance          6   0 /proc/irq/8/smp_affinity
617    irqbalance          6   0 /proc/irq/12/smp_affinity
617    irqbalance          6   0 /proc/irq/14/smp_affinity
617    irqbalance          6   0 /proc/irq/15/smp_affinity
804    vminfo              6   0 /var/run/utmp
598    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
598    dbus-daemon        18   0 /usr/share/dbus-1/system-services
598    dbus-daemon        -1   2 /lib/dbus-1/system-services
598    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
```

6. Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.

Ответ: uname() returns system information in the structure pointed to by buf.

    Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
```bash
vagrant@vagrant:~$ sudo cat /proc/sys/kernel/{ostype,hostname,osrelease,version,domainname}
Linux
vagrant
5.4.0-80-generic
#90-Ubuntu SMP Fri Jul 9 22:49:44 UTC 2021
(none) 
```


7. Чем отличается последовательность команд через ; и через && в bash? Например:

```shell
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```

Есть ли смысл использовать в bash &&, если применить set -e?

Ответ: Команды, разделенные ;, выполняются последовательно независимо от их статуса завершения.

С && вторая команда выполняется только в том случае, если первая завершается успешно (возвращает статус выхода 0). 

Если установлен set -e, использовать && нет смысла, для себя решил, что set -e для скриптов, а && для однострочников напрямую из командной строки (удобно, компактно, повышает надежность и безопасность однострочника).

8. Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?

Ответ: Данный набор опций позволит получать более надежные и безопасные сценарии, а также сделает их отладку удобнее.

    -e скрипт немедленно завершит работу, если любая команда выйдет с ошибкой. По-умолчанию, игнорируются любые неудачи и сценарий продолжет выполнятся.
    -u проверяет инициализацию переменных в скрипте. Если переменной не будет, скрипт немедленно завершится. Данный параметр достаточно умен, чтобы нормально работать с переменной по-умолчанию ${MY_VAR:-$DEFAULT} и условными операторами (if, while, и др).
    -x выводит на экран трассировку простых команд, for, case, select и арифметических for и их аргументов или связанных списков слов после того, как они развернуты и до их выполнения. Значение переменной PS4 раскрывается, а результирующее значение выводится перед командой и ее расширенными аргументами. Стоит учитывать, что все переменные будут уже раскрыты, и с этим нужно быть аккуратнее, к примеру если в скрипте используются пароли.
    -o pipefail позволяет убедиться, что все команды в пайпах завершились успешно. Без этой опции Bash возвращает только код ошибки последней команды в пайпе (конвейере). И параметр -e проверяет только его. 


9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

Ответ: В выводе ps -o stat очень мало процессов (статусов), поэтому запустил ps -axo stat. Чаще всего встречаются статусы с кодом S.
```bash
vagrant@vagrant:~$ ps -axo stat | tail -n +2 | while read str; do echo ${str:0:1}; done | sort | uniq -c | sort -n -k1 -r
     57 S
     52 I
      1 R
```

```shell
PROCESS STATE CODES
       Here are the different values that the s, stat and state output specifiers (header "STAT" or "S") will display to describe the state of a process:

               D    uninterruptible sleep (usually IO)
               I    Idle kernel thread
               R    running or runnable (on run queue)
               S    interruptible sleep (waiting for an event to complete)
               T    stopped by job control signal
               t    stopped by debugger during the tracing
               W    paging (not valid since the 2.6.xx kernel)
               X    dead (should never be seen)
               Z    defunct ("zombie") process, terminated but not reaped by its parent

       For BSD formats and when the stat keyword is used, additional characters may be displayed:

               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group

```
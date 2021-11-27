#Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда cd? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей,
если считаете что она могла бы быть другого типа.

Ответ:
```text
Команда cd это встроенная команда Bash и меняет текущую папку только для оболочки, в которой 
выполняется. Ее нет в файловой системе. Таким образом команда является частью оболочки. Текущий каталог
является свойством каждого процесса, bash запускает процессы, у каждого свой текущий каталог. Запуски 
других процессов (вроде внешней программы cd) не повлияют на текущий каталог самого процесса bash.  
```

2. Какая альтернатива без pipe команде grep <some_string> <some_file> | wc -l? man grep поможет в ответе 
на этот вопрос. Ознакомьтесь с документом о других подобных некорректных вариантах использования pipe.

    Ответ:
 ```shell
vagrant@vagrant:~$ cat /usr/share/awk/ctime.awk
# ctime.awk
#
# awk version of C ctime(3) function

function ctime(ts,    format)
{
    format = "%a %b %e %H:%M:%S %Z %Y"

    if (ts == 0)
        ts = systime()       # use current time as default
    return strftime(format, ts)
}
vagrant@vagrant:~$ grep format /usr/share/awk/ctime.awk | wc -l
3
vagrant@vagrant:~$ grep -c format /usr/share/awk/ctime.awk 
3
vagrant@vagrant:~$ awk '/format/ {++x} END {print x}' /usr/share/awk/ctime.awk 
3
```

3. Какой процесс с PID 1 является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

    Ответ:
```shell
vagrant@vagrant:~$ ps aux | head -n 2
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.9 167332 11232 ?        Ss   22:07   0:00 /sbin/init
vagrant@vagrant:~$ pstree -H 1
systemd─┬─VBoxService───8*[{VBoxService}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─agetty
        ├─atd
        ├─cron
        ├─dbus-daemon
        ├─irqbalance───{irqbalance}
        ├─multipathd───6*[{multipathd}]
        ├─networkd-dispat
        ├─polkitd───2*[{polkitd}]
        ├─rpcbind
        ├─rsyslogd───3*[{rsyslogd}]
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-network
        ├─systemd-resolve
        └─systemd-udevd
```

4. Как будет выглядеть команда, которая перенаправит вывод stderr ls на другую сессию терминала?

    Ответ:
```shell
agrant@vagrant:~$ who -a
           system boot  2021-11-26 22:07
           run-level 5  2021-11-26 22:07
LOGIN      tty1         2021-11-26 22:07              1915 id=tty1
vagrant  + pts/0        2021-11-26 22:07   .         12002 (10.0.2.2)
vagrant@vagrant:~$ screen -d -m -S vagrant
vagrant@vagrant:~$ who -a
           system boot  2021-11-26 22:07
           run-level 5  2021-11-26 22:07
LOGIN      tty1         2021-11-26 22:07              1915 id=tty1
vagrant  + pts/0        2021-11-26 22:07   .         12002 (10.0.2.2)
           pts/1        2021-11-26 22:57             12825 id=ts/1  term=0 exit=0
vagrant@vagrant:~$ ls /root/ 2>/dev/pts/1
vagrant@vagrant:~$ screen -x vagrant
vagrant@vagrant:~$ ls: cannot open directory '/root/': Permission denied
```

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите
работающий пример.

    Ответ: Да.
```shell
vagrant@vagrant:~$ cat /tmp/test_file.txt
123
1223
1e223
1e2f23
vagrant@vagrant:~$ cat /tmp/test_file1.txt
cat: /tmp/test_file1.txt: No such file or directory
vagrant@vagrant:~$ cat < /tmp/test_file.txt > /tmp/test_file1.txt
vagrant@vagrant:~$ cat /tmp/test_file1.txt
123
1223
1e223
1e2f23
vagrant@vagrant:~$
```   
6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY?
Сможете ли вы наблюдать выводимые данные?

    Ответ: 
```text
Отправить получится. Прочитать, только переключившись на соответствующий tty.
Чтобы иметь возможность пользоваться сессией и из графического режима, и из консоли 
аппаратного эмулятора, надо запустить ее, например в мультиплексоре screen (pts/9). 
```
```shell
dmitriy@dellix:~$ tty
/dev/pts/8
dmitriy@dellix:~$ who -a
           загрузка системы 2021-11-26 21:38
           уровень выполнения 5 2021-11-26 21:38
dmitriy  ? :1           2021-11-26 21:39   ?          3409 (:1)
dmitriy  - tty3         2021-11-27 03:27 00:02       27383
           pts/9        2021-11-27 03:44             29728 id=ts/9  терминал=0 выход=0
dmitriy@dellix:~$ echo "12345" > /dev/tty3
dmitriy@dellix:~$ tty
/dev/tty3
dmitriy@dellix:~$ 12345
```
7. Выполните команду bash 5>&1. К чему она приведет? Что будет, если вы выполните 
echo netology > /proc/$$/fd/5? Почему так происходит?

    Ответ: 
```test 
Первая команда создала новый дескриптор 5 и перенаправила его в stdout, вторая команда 
отправила в него строку "netology", которую мы получили в stdout  на /dev/pts/0.
и получили в своей сессии терминала
```
```shell
agrant@vagrant:~$ bash 5>&1
vagrant@vagrant:~$ echo netology > /proc/$$/fd/5
netology
vagrant@vagrant:~$ ls -l /proc/$$/fd/
total 0
lrwx------ 1 vagrant vagrant 64 Nov 27 01:15 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Nov 27 01:15 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Nov 27 01:15 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Nov 27 01:15 255 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Nov 27 01:11 5 -> /dev/pts/0
```

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв 
при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout 
команды слева от | на stdin команды справа. Это можно сделать, поменяв стандартные потоки местами 
через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

    Ответ:
```shell
vagrant@vagrant:~$ ls -l /root 7>&2 2>&1 1>&7
ls: cannot open directory '/root': Permission denied

7>&2 - новый дескриптор перенаправили в stderr
2>&1 - stderr перенаправили в stdout 
1>&7 - stdout - перенаправили в в новый дескриптор
```

9. Что выведет команда cat /proc/$$/environ? Как еще можно получить аналогичный по содержанию вывод?

    Ответ: 
```text
Эта команда выводит переменные окружения, аналогичный вывод дают env и printenv,
но с разделением переменных по строкам. tr -d '\n' удаляет символ конца строки и вывод команд 
становится идентичным, за исключением последней переменной "_", которая отображает вывод последней 
запущенной команды. 
```
```shell
vagrant@vagrant:~$ cat /proc/$$/environ
SHELL=/bin/bashLANGUAGE=en_US:PWD=/home/vagrantLOGNAME=vagrantXDG_SESSION_TYPE=ttyMOTD_SHOWN=pamHOME=/home/vagrantLANG=en_US.UTF-8LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:SSH_CONNECTION=10.0.2.2 41626 10.0.2.15 22LESSCLOSE=/usr/bin/lesspipe %s %sXDG_SESSION_CLASS=userTERM=xterm-256colorLESSOPEN=| /usr/bin/lesspipe %sUSER=vagrantSHLVL=1XDG_SESSION_ID=4XDG_RUNTIME_DIR=/run/user/1000SSH_CLIENT=10.0.2.2 41626 22XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktopPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/binDBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/busSSH_TTY=/dev/pts/0_=/usr/bin/bash
agrant@vagrant:~$ printenv | tr -d '\n'
SHELL=/bin/bashLANGUAGE=en_US:PWD=/home/vagrantLOGNAME=vagrantXDG_SESSION_TYPE=ttyMOTD_SHOWN=pamHOME=/home/vagrantLANG=en_US.UTF-8LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:SSH_CONNECTION=10.0.2.2 41626 10.0.2.15 22LESSCLOSE=/usr/bin/lesspipe %s %sXDG_SESSION_CLASS=userTERM=xterm-256colorLESSOPEN=| /usr/bin/lesspipe %sUSER=vagrantSHLVL=1XDG_SESSION_ID=4XDG_RUNTIME_DIR=/run/user/1000SSH_CLIENT=10.0.2.2 41626 22XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktopPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/binDBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/busSSH_TTY=/dev/pts/0_=/usr/bin/printenv
vagrant@vagrant:~$ env | tr -d '\n'
SHELL=/bin/bashLANGUAGE=en_US:PWD=/home/vagrantLOGNAME=vagrantXDG_SESSION_TYPE=ttyMOTD_SHOWN=pamHOME=/home/vagrantLANG=en_US.UTF-8LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:SSH_CONNECTION=10.0.2.2 41626 10.0.2.15 22LESSCLOSE=/usr/bin/lesspipe %s %sXDG_SESSION_CLASS=userTERM=xterm-256colorLESSOPEN=| /usr/bin/lesspipe %sUSER=vagrantSHLVL=1XDG_SESSION_ID=4XDG_RUNTIME_DIR=/run/user/1000SSH_CLIENT=10.0.2.2 41626 22XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktopPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/binDBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/busSSH_TTY=/dev/pts/0_=/usr/bin/env
```

10. Используя man, опишите что доступно по адресам /proc/<PID>/cmdline, /proc/<PID>/exe.

    Ответ:
```shell
root@dellix:~# cat /proc/4026/cmdline 
/usr/bin/gjs/usr/share/gnome-shell/org.gnome.ScreenSaver

root@dellix:~# ls -lh /proc/4026/exe 
lrwxrwxrwx 1 dmitriy dmitriy 0 ноя 26 21:39 /proc/4026/exe -> /usr/bin/gjs-console

man proc
/proc/[pid]/cmdline
              This read-only file holds the complete command line for the process, unless the process is a zombie.  In the latter case, there is nothing in this file: that is, a read  on  this  file
              will return 0 characters.  The command-line arguments appear in this file as a set of strings separated by null bytes ('\0'), with a further null byte after the last string.
/proc/[pid]/exe
              Under  Linux 2.2 and later, this file is a symbolic link containing the actual pathname of the executed command.  This symbolic link can be dereferenced normally; attempting to open it
              will open the executable.  You can even type /proc/[pid]/exe to run another copy of the same executable that is being run by process [pid].  If the pathname has been unlinked, the sym‐
              bolic  link  will contain the string '(deleted)' appended to the original pathname.  In a multithreaded process, the contents of this symbolic link are not available if the main thread
              has already terminated (typically by calling pthread_exit(3)).
              
/proc/[pid]/cmdline - полный путь до исполняемого файла процесса [pid]
/proc/[pid]/exe - является символической ссылкой на файл запущенного для процесса [pid], 
                        cat выведет содержимое запущенного файла, 
                        запуск этого файла,  запустит еще одну копию самого файла 
```

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор 
с помощью /proc/cpuinfo.

    Ответ:
```shell
vagrant@vagrant:~$ cat /proc/cpuinfo | grep sse
...... sse4_2 .....
```
12. При открытии нового окна терминала и vagrant ssh создается новая сессия и выделяется pty. Это можно
подтвердить командой tty, которая упоминалась в лекции 3.2. Однако:

 vagrant@netology1:~$ ssh localhost 'tty'
 not a tty
 Почитайте, почему так происходит, и как изменить поведение.

    Ответ: 
```text
Многие команды для своего выполнения не требуют терминала. 
```
```shell
vagrant@vagrant:~$ ssh localhost 'ip a'
vagrant@localhost's password: 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:73:60:cf brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 83832sec preferred_lft 83832sec
    inet6 fe80::a00:27ff:fe73:60cf/64 scope link 
       valid_lft forever preferred_lft forever
```
```test
Но команде tty требуется запущеный терминал, чтобы сообщить его название, а ssh при запуске с 
удаленной командой не создает TTY, это позволяет передавать двоичные данные и т. Д., Не 
сталкиваясь с причудами TTY. Если удаленной команде требуется TTY, как в нашем случае, 
надо использовать опцию -t, тогда ssh принудительно запустит псевдотерминал.
```
```shell
vagrant@vagrant:~$ ssh -t localhost 'tty'
vagrant@localhost's password: 
/dev/pts/1
Connection to localhost closed.
```

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте
сделать это, воспользовавшись reptyr. Например, так можно перенести в screen процесс, который вы 
запустили по ошибке в обычной SSH-сессии.

    Ответ: 
```text
Порядок действий: запускаем в терминале например программу top. Затем создаем в sceen
сессию, в ней запускаем команду ps aux, находим pid программы top и указываем его в 
качестве параметра программе reptyr, программа еtop начинает выполняться в сессии в screen, а 
в предыдущем терминале выполнение программы топ завершается с сообщением 
"[1]+  Остановлен    top" 
```    


14. sudo echo string > /root/new_file не даст выполнить перенаправление под обычным пользователем, 
так как перенаправлением занимается процесс shell'а, который запущен без sudo под вашим 
пользователем. Для решения данной проблемы можно использовать конструкцию 
echo string | sudo tee /root/new_file. Узнайте что делает команда tee и почему в отличие от 
sudo echo команда с sudo tee будет работать.

    Ответ:
```text
команда tee делает вывод одновременно и в файл, указаный в качестве параметра, и в stdout.
В данном примере команда tee, запущенная sudo получает вывод из stdin, перенаправленный через pipe от 
stdout команды echo и записывает его (вывод) в файл, к которому умеет доступ (sudo). 
```

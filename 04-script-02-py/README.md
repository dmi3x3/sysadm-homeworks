### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-02-py/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ                                                                 |
| ------------- |-----------------------------------------------------------------------|
| Какое значение будет присвоено переменной `c`?  | Никакого. Python не умеет складывать  с числа (int) со строками (str) |
| Как получить для переменной `c` значение 12?  | с = str(a) + b                                                        |
| Как получить для переменной `c` значение 3?  | c = a + int(b)                                                        |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

git_dir_path = "~/netology/sysadm-homeworks"
true_path = os.path.normpath(os.path.abspath(os.path.expanduser(os.path.expandvars(git_dir_path))))
bash_command = [f"cd {true_path}", "git status --short"]
result_os = os.popen(' && '.join(bash_command)).read()
spisok_result = filter(None, result_os.split('\n'))
for result in spisok_result:
    if result[1] == 'M':
        prepare_result = result[1:].replace('M ', '')
        print(os.path.join(true_path, prepare_result))
```

### Вывод скрипта при запуске при тестировании:
```
/home/dmitriy/netology/sysadm-homeworks_py/test_dir/file1
/home/dmitriy/netology/sysadm-homeworks_py/test_dir/file2
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os
import sys

git_dir_path = sys.argv[1]
bash_command = 0
true_path = os.path.normpath(os.path.abspath(git_dir_path))

if os.path.exists(true_path):
    if os.path.isdir(true_path):
        if os.path.isdir(true_path+'/.git'):
            bash_command = [f"cd {true_path}", "git status --short"]
        else:
            print(f'\033[31m Каталог \'{true_path}\' не является git-репозиторием')
            exit()
    else:
        print(f'\033[31m Объект \'{true_path}\' существует, но не является каталогом')
        exit()
else:
    print(f'\033[31m Каталог \'{true_path}\' не найден')
    exit()

result_os = os.popen(' && '.join(bash_command)).read()
spisok_result = filter(None, result_os.split('\n'))


for result in spisok_result:
    if result[1] == 'M':
        prepare_result = result[1:].replace('M ', '')
        print(os.path.join(true_path, prepare_result))

```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py /home/vagrant/netology/sysadm-homeworks
/home/vagrant/netology/sysadm-homeworks/test_dir/file1
/home/vagrant/netology/sysadm-homeworks/test_dir/file2
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py /home/${USER}/netology/sysadm-homeworks
/home/vagrant/netology/sysadm-homeworks/test_dir/file1
/home/vagrant/netology/sysadm-homeworks/test_dir/file2
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py ~/netology/sysadm-homeworks
/home/vagrant/netology/sysadm-homeworks/test_dir/file1
/home/vagrant/netology/sysadm-homeworks/test_dir/file2
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py ../../netology/sysadm-homeworks
/home/vagrant/netology/sysadm-homeworks/test_dir/file1
/home/vagrant/netology/sysadm-homeworks/test_dir/file2
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py /netology/sysadm-homeworks_py
Каталог '/netology/sysadm-homeworks_py' не найден
vagrant@vagrant:~/netology/python_scripts$ ./test_python2.py /tmp/res_py.txt 
Объект '/tmp/res_py.txt' существует, но не является каталогом
vagratn@vagrant:~/netology/python_scripts$ ./test_python2.py /tmp/
Каталог '/tmp' не является git-репозиторием
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket
 
def uptime_bot(hosts):
    while True:
        for host in hosts.keys():
            cur_ip = hosts[host]["ipv4"]
            check_ip = socket.gethostbyname(host)
            if check_ip != cur_ip:
                print(f"\033[31m [ERROR] \"{host}\" IP mismatch: \"{cur_ip}\" \"{check_ip}\"")
                hosts[host]["ipv4"] = check_ip
            else:
                print(f"\033[35m {host.ljust(20)} -      {cur_ip}")
        time.sleep(3)
 
if __name__ == '__main__':
    hosts_serv = {"drive.google.com": {"ipv4": "0.0.0.0"}, "mail.google.com": {"ipv4": "0.0.0.0"}, "google.com": {"ipv4": "0.0.0.0"}}
    uptime_bot(hosts_serv)
```

### Вывод скрипта при запуске при тестировании:
```
 [ERROR] "drive.google.com" IP mismatch: "0.0.0.0" "64.233.162.194"
 [ERROR] "mail.google.com" IP mismatch: "0.0.0.0" "142.251.1.18"
 [ERROR] "google.com" IP mismatch: "0.0.0.0" "64.233.164.138"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.18" "142.251.1.17"
 [ERROR] "google.com" IP mismatch: "64.233.164.138" "64.233.164.113"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.17" "142.251.1.83"
 [ERROR] "google.com" IP mismatch: "64.233.164.113" "64.233.164.100"
 drive.google.com     -      64.233.162.194
 mail.google.com      -      142.251.1.83
 [ERROR] "google.com" IP mismatch: "64.233.164.100" "74.125.205.102"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.83" "142.251.1.18"
 [ERROR] "google.com" IP mismatch: "74.125.205.102" "74.125.205.139"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.18" "142.251.1.17"
 [ERROR] "google.com" IP mismatch: "74.125.205.139" "74.125.205.138"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.17" "142.251.1.19"
 google.com           -      74.125.205.138
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.19" "142.251.1.18"
 [ERROR] "google.com" IP mismatch: "74.125.205.138" "74.125.205.139"
 drive.google.com     -      64.233.162.194
 [ERROR] "mail.google.com" IP mismatch: "142.251.1.18" "142.251.1.83"
 google.com           -      74.125.205.139
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так получилось, что мы очень часто вносим правки в конфигурацию своей системы прямо на сервере. Но так как вся наша команда разработки держит файлы конфигурации в github и пользуется gitflow, то нам приходится каждый раз переносить архив с нашими изменениями с сервера на наш локальный компьютер, формировать новую ветку, коммитить в неё изменения, создавать pull request (PR) и только после выполнения Merge мы наконец можем официально подтвердить, что новая конфигурация применена. Мы хотим максимально автоматизировать всю цепочку действий. Для этого нам нужно написать скрипт, который будет в директории с локальным репозиторием обращаться по API к github, создавать PR для вливания текущей выбранной ветки в master с сообщением, которое мы вписываем в первый параметр при обращении к py-файлу (сообщение не может быть пустым). При желании, можно добавить к указанному функционалу создание новой ветки, commit и push в неё изменений конфигурации. С директорией локального репозитория можно делать всё, что угодно. Также, принимаем во внимание, что Merge Conflict у нас отсутствуют и их точно не будет при push, как в свою ветку, так и при слиянии в master. Важно получить конечный результат с созданным PR, в котором применяются наши изменения. 

### Ваш скрипт:
```python
???
```

### Вывод скрипта при запуске при тестировании:
```
???
```

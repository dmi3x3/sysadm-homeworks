### Как сдавать задания

Вы уже изучили блок «Системы управления версиями», и начиная с этого занятия все ваши работы будут приниматься ссылками на .md-файлы, размещённые в вашем публичном репозитории.

Скопируйте в свой .md-файл содержимое этого файла; исходники можно посмотреть [здесь](https://raw.githubusercontent.com/netology-code/sysadm-homeworks/devsys10/04-script-03-yaml/README.md). Заполните недостающие части документа решением задач (заменяйте `???`, ОСТАЛЬНОЕ В ШАБЛОНЕ НЕ ТРОГАЙТЕ чтобы не сломать форматирование текста, подсветку синтаксиса и прочее, иначе можно отправиться на доработку) и отправляйте на проверку. Вместо логов можно вставить скриншоты по желани.

# Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" : [
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import time
import socket
import json
import yaml

log_file = "./services.log"


def uptime_bot(hosts):
    while True:
        for host in hosts:
            cur_ip = hosts[host]
            check_ip = socket.gethostbyname(host)
            hosts_f = {"0":"0.0.0.0"}
            if check_ip != cur_ip:
                print(f"\033[31m [ERROR] \"{host}\" IP mismatch: \"{cur_ip}\" \"{check_ip}\"")
                with open(log_file,"a") as fl:
                    print(f"[ERROR] \"{host}\" IP mismatch: \"{cur_ip}\" \"{check_ip}\"",file=fl)
                hosts[host] = check_ip
                hosts_f = {host:check_ip}
                write_jy_format(hosts_f,host)
            else:
                print(f"\033[35m {host.ljust(20)} -      {cur_ip}")
                with open(log_file,"a") as fl:
                    print(f"{host.ljust(20)} -      {cur_ip}",file=fl)
                hosts_f = {host:check_ip}
                write_jy_format(hosts_f,host)
        time.sleep(3)


def write_jy_format(hosts,host):
    with open("./service_"+host+".json", 'w+') as fj:
        fj.write(json.dumps(hosts))
    with open("./service_"+host+".yaml", 'w+') as fy:
        fy.write(yaml.dump([hosts]))

if __name__ == '__main__':
    hosts_serv = {"drive.google.com": "0.0.0.0", "mail.google.com": "0.0.0.0", "google.com": "0.0.0.0"}
    try:
        uptime_bot(hosts_serv)
    except KeyboardInterrupt as err:
        print('Пользователь завершил выполнение программы', err)
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@vagrant:~/netology/python_scripts$ ./services_mon.py
 [ERROR] "drive.google.com" IP mismatch: "0.0.0.0" "209.85.233.194"
 [ERROR] "mail.google.com" IP mismatch: "0.0.0.0" "209.85.233.83"
 [ERROR] "google.com" IP mismatch: "0.0.0.0" "74.125.131.139"
 drive.google.com     -      209.85.233.194
 mail.google.com      -      209.85.233.83
 [ERROR] "google.com" IP mismatch: "74.125.131.139" "74.125.131.100"
 drive.google.com     -      209.85.233.194
 [ERROR] "mail.google.com" IP mismatch: "209.85.233.83" "209.85.233.17"
 [ERROR] "google.com" IP mismatch: "74.125.131.100" "74.125.131.113"
 drive.google.com     -      209.85.233.194
 [ERROR] "mail.google.com" IP mismatch: "209.85.233.17" "209.85.233.19"
 [ERROR] "google.com" IP mismatch: "74.125.131.113" "74.125.131.100"
 drive.google.com     -      209.85.233.194
 [ERROR] "mail.google.com" IP mismatch: "209.85.233.19" "209.85.233.83"
 [ERROR] "google.com" IP mismatch: "74.125.131.100" "74.125.131.139"
 drive.google.com     -      209.85.233.194
 [ERROR] "mail.google.com" IP mismatch: "209.85.233.83" "209.85.233.18"
 [ERROR] "google.com" IP mismatch: "74.125.131.139" "74.125.131.101"
^CПользователь завершил выполнение программы
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
vagrant@vagrant:~/netology/python_scripts$ find ./service_*.json | while read str; do echo -e "$str \n $(cat $str)";done
./service_drive.google.com.json 
 {"drive.google.com": "209.85.233.194"}
./service_google.com.json 
 {"google.com": "74.125.131.101"}
./service_mail.google.com.json 
 {"mail.google.com": "209.85.233.18"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
vagrant@vagrant:~/netology/python_scripts$ find ./service_*.yaml | while read str; do echo -e "$str \n $(cat $str)";done
./service_drive.google.com.yaml 
 - drive.google.com: 209.85.233.194
./service_google.com.yaml 
 - google.com: 74.125.131.101
./service_mail.google.com.yaml 
 - mail.google.com: 209.85.233.18
```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Так как команды в нашей компании никак не могут прийти к единому мнению о том, какой формат разметки данных использовать: JSON или YAML, нам нужно реализовать парсер из одного формата в другой. Он должен уметь:
   * Принимать на вход имя файла
   * Проверять формат исходного файла. Если файл не json или yml - скрипт должен остановить свою работу
   * Распознавать какой формат данных в файле. Считается, что файлы *.json и *.yml могут быть перепутаны
   * Перекодировать данные из исходного формата во второй доступный (из JSON в YAML, из YAML в JSON)
   * При обнаружении ошибки в исходном файле - указать в стандартном выводе строку с ошибкой синтаксиса и её номер
   * Полученный файл должен иметь имя исходного файла, разница в наименовании обеспечивается разницей расширения файлов

### Ваш скрипт:
```python
???
```

### Пример работы скрипта:
???
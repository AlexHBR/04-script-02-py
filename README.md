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
| Вопрос  | Ответ                  |
| ------------- |------------------------|
| Какое значение будет присвоено переменной `c`?  | ошибка сложения  типов |
| Как получить для переменной `c` значение 12?  | c = str(a) + b         |
| Как получить для переменной `c` значение 3?  | c = a + int(b)         |

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
bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
dir_name = os.getcwd()+bash_command[0][4:]+'/'
for result in result_os.split('\n'):
    if result.find('изменено') != -1 or result.find('modified') != -1 :
        prepare_result = result.replace('\tmodified:', '').replace('\tизменено:', '').lstrip()
        print(dir_name + prepare_result)
        is_change = True
        #break -выхиод при наличии первого измененого файла убираем
if not is_change : print('No modified files git')
```

### Вывод скрипта при запуске при тестировании:
```
pp@compn:~/laba$ ./1.py
/home/pp/laba/netology/sysadm-homeworks/04-script-03-yaml/additional-info/README.md
/home/pp/laba/netology/sysadm-homeworks/README.md
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import os
import sys
if len(sys.argv) == 1 :
    dir_inp = "~/netology/sysadm-homeworks"
else :
    dir_inp =sys.argv[1]
#Без параметров проверям по умолчанию ~/netology/sysadm-homeworks
bash_command = ["cd "+str(dir_inp),"git rev-parse --show-toplevel", "git status --porcelain"]
result_os = os.popen(' && '.join(bash_command)).read()
dir_name = result_os.split('\n')[0]+'/'
for result in result_os.split('\n'):
    if result[0:3] == 'AM ' :  print(dir_name + result[3:])
#В случае отсутствия директории или репозитария будет вывод ошибок системы. По условию не сказанно, что делать если указана поддиректория git, в данном алгоритме выводятся все изменения с корня репозитория.
```

### Вывод скрипта при запуске при тестировании:
```
Вывод если нет указанной директории
pp@compn:~/laba$ ./1.py /home/pp3
/bin/sh: 1: cd: can't cd to /home/pp3

Вывод если нет репозиория в директории
pp@compn:~/laba$ ./1.py /home/pp/netology/
fatal: не найден git репозиторий (или один из родительских каталогов): .git

Вывод по умолчанию проверяем ~/netology/sysadm-homeworks
pp@compn:~/laba$ ./1.py
/home/pp/netology/sysadm-homeworks/04-script-03-yaml/additional-info/README.md
/home/pp/netology/sysadm-homeworks/README.md

Вывод из поддиректории git /home/pp/netology/sysadm-homeworks/04-script-03-yaml
pp@compn:~/laba$ ./1.py /home/pp/netology/sysadm-homeworks/04-script-03-yaml
/home/pp/netology/sysadm-homeworks/04-script-03-yaml/additional-info/README.md
/home/pp/netology/sysadm-homeworks/README.md


```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
import socket, time


def ip_ch(name_service):
    try:
        return socket.gethostbyname(name_service)
    except Exception as error:
        return ("Error connect" + str(error))


mas = ['drive.google.com', 'mail.google.com', 'google.com']
ip = [ip_ch(mas[0]), ip_ch(mas[1]), ip_ch(mas[2])]
for c, n in enumerate(mas): print(n, ip[c])
a = 0
while a < 10:
    for c, n in enumerate(mas):
        newip = ip_ch(n)
        if ip[c] != newip:
            print('[ERROR] ' + n + ' mismatch: ' + ip[c] + ' ' + newip)
        ip[c] = newip
        # print(n, ip[c]) #можно организовать вывод но будет моного лишних данных
    time.sleep(60)
```

### Вывод скрипта при запуске при тестировании:
```
drive.google.com 74.125.205.194
mail.google.com 142.251.1.19
google.com 142.251.1.102
[ERROR] google.com mismatch: 142.251.1.102 64.233.162.113

алгоритм выведет ошибку если будет отсутвовать доступ по новому адресу
если для эмуляции ошибки исправить mail.google.com1 то вывод будет следующий
drive.google.com 74.125.205.194
mail.google.com1 Error connect[Errno 11001] getaddrinfo failed
google.com 64.233.162.113

```

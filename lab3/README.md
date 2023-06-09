# Создание ansible скрипта для создания распределителя нагрузки на haproxy + keepalived
В данном проекте создан ansible сценарий, который настраивает работу веб-серверов и распределителя нагрузки. В этой лабораторной работе в роли распределителя нагрузки выступает ПО haproxy с повышенной отказаустойчивостью в виде keepalived (этот модуль необходим для настройки виртуального ip-адреса).

Для подробного ознакомления с параметрами проекта, необходимо прочитать про каждый файл подробнее.

## Файлы в проекте
## *1. VagrantFile*
Этот файл поднимает виртуальные машины, необходимые для проверки скрипта.

Не влияет на работу haproxy.yml
## *2. Inventory-файл*
Основной конфигурационный файл. 
В нем есть две группы:
* webservers

    Содержит в себе имя хоста первым параметром и ip-адрес сервера, который на себе будет содержать html страницу.
* balancer

    Содержит в себе имя хоста первым параметром и ip-адрес сервера, который на себе будут содержать балансировщик трафика соответвественно. Также обязательным атрибутом выступает параметр "приоритет". Он необходим для дальнейшей настройки VIP (virtual ip).


## *3. ansible.cfg*
Содержит в себе следующие параметры:
1. Указываем inventory файл
2. Указываем пользователя, с которомы мы работаем
3. Нужно ли осуществлять host_key_checking
4. Парметр defualt_transport. (параемтр smart ставит автоматическое переключение ммежду ssh и paramiko в зависимости от ОС)


## *4. nginx.yml*
Является основным основным playbook-файлом для установки ПО на сервера.

Содержит в себе запуск 3 ролей, о которых будет прописано далее. Каждая из них запускается в зависимости от принадлежности хоста одной из групп. Disabled selinux является общей, поэтому

запускается внезависимости от групп, так как является обязательной настройкой системы.

# **Папка Roles**
Папка с основными скриптами. Находится несколько ролей, о каждой 
## 1. selinux-disabled
Общая настройка всех устройств в этой схеме. Отключает системную защиту, препядствующую работе серверов и перезагружает сервер.
Результат для трех пк: 3 changed

## 2. balancer
Настраивает сервера-балансировщиков.

Устанавливает nginx и меняет его конфигурационный файл, настраивая балансировщик в режим round robin. Шаблон конфигурационного файла лежит в папке templates

Результат 1 success, 1 changed

## 3. webservers
Настраивает веб-сервера. Устанавливает на них nginx, изменяет конфигурационный файл и меняет html-документ, подгружающийся на этот сервер.
Для наглядности html-документ изменен таким образом, чтобы увидеть работоспособность распределителя нагрузки.

Запуск и использования ansible playbook
======================================
## 1. inventory
Первым этапом необходимо изменить inventory-файл. В него внести ip-адреса и имена хостов.
## 2. Шаблоны
Этот этап не является обязательным, однако для работы его необходимо учитывать. 

В /roles/webservers/templates необходимо изменть шаблон html-документа (файл index.html.j2).
## 3. Запуск playbook
Если необходимо полностью установить все компоненты, описанные в ролях, нужно написать в терминале 
"ansible-playbook haproxy.yml"
Эта команда выполнит все task файлы из всех ролей. 

### Теги
Не всегда некоторые параметры являются необходимыми. Для этого написаны специальные теги, по которым можно запустить отдельный элемент.

#### 1. html

Этот тег меняет конфигурацонный файл nginx на веб-серверах

#### 2. balancer

Выполняет настройку серверов-балансировщиков

#### 3. selinux

Выполняет отключение selinux на всех серверах


# **Результат**
В результате мы настраиваем веб-сервера и балансировщик.

 Для наглядности было создано два разных сайта, чтобы увидеть работаспособность скрипта.
![](web1.jpg)
![](web2.jpg)
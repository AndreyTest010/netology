# Домашнее задание к занятию "Disaster recovery и Keepalived" - `Корнилов Андрей`



### Задание 1

Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.
На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

---
схема в формате pkt
https://github.com/AndreyTest010/netology/blob/main/DisasterRecovery/hsrp_advanced.pkt

![cisco](https://github.com/AndreyTest010/netology/blob/main/DisasterRecovery/screen/first.png)




### Задание 2
Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.
Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

Ход работы:

bash скрипт


#!/bin/bash
WEB_SERVER="localhost"    
PORT=80                   
INDEX_FILE="/var/www/html/index.html"  
if curl -s --head --connect-timeout 2 http://$WEB_SERVER:$PORT | grep "200 OK" > /dev/null && [ -f "$INDEX_FILE" ]; then
    exit 0  
else
    exit 1  
fi

--------------------------------------------------------------------------------------------------------------------------
keepalived конфигурационный файл

vrrp_track_process check_nginx {
       process "nginx"
}
vrrp_instance VI_1 {
        state MASTER
        interface ens33
        virtual_router_id 15
        priority 255
        advert_int 1


        virtual_ipaddress {
              192.168.111.15/24
        }

     
        track_process {
	           check_nginx

		}
}
vrrp_script chk_web_server {
    script "/usr/local/bin/check_web_server.sh"
    interval 3
    weight -2
}

-------------------------------------------------------------------------------------------------------------------------

Скриншоты

![vm1](https://github.com/AndreyTest010/netology/blob/main/DisasterRecovery/screen/keep1.jpg)
![vm2](https://github.com/AndreyTest010/netology/blob/main/DisasterRecovery/screen/keep2.jpg)



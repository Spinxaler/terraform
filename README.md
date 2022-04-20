
# Домашнее задание к занятию "5.3. 

## Обязательная задача 1

https://hub.docker.com/u/spinxaler

## Обязательная задача 2
Сценарий:

Высоконагруженное монолитное java веб-приложение;
Физическая машина, для максимальной нагрузки на ресурсы.

Nodejs веб-приложение;
Докер подойдёт так как легко менять версию и не требовательно к ресурсам

Мобильное приложение c версиями для Android и iOS;
Докер не подойдёт, так как это разный тип приложений, а докер не позволяет использовать разные ядра, Лучше будет использовать разные виртуальный машины.

Шина данных на базе Apache Kafka;
Могу основыватся только на том что Apache Kafka есть готовые образы и скачиваний очень много, то его в докер используют и скорее всего он подходит. 

Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
Можно использовать докер, приложения не нагруженые и не так кретичны к потери части данных. На сайте это рекомендованный способ установки.

Мониторинг-стек на базе Prometheus и Grafana;
Докер подойдёт, простая и быстрая настройка.

MongoDB, как основное хранилище данных для java-приложения;
Лучше использовать виртуальную машину, или физическую для того чтобы не потерять данные. 

Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
Docker является одним из рекомендованных разработчиками способов установки, следовательно подходит.

## Обязательная задача 3
'''
root@spinxaler-VirtualBox:~# docker run --name my_centos -d -v /home/spinxaler/data:/data centos  sleep 10000
e70de35d058ed1bd64ba96e2289d602a6ae6b0c6d0ed11679a892b94a5f648de
root@spinxaler-VirtualBox:~# docker run --name my_debian -d -v /home/spinxaler/data:/data debian  sleep 10000
5c52b6a8e41417e71f8b5bc3fd80c87e947c529d0ff714ed95255b9ae856819c
root@spinxaler-VirtualBox:~# docker ps
CONTAINER ID   IMAGE     COMMAND         CREATED          STATUS          PORTS     NAMES
5c52b6a8e414   debian    "sleep 10000"   8 seconds ago    Up 7 seconds              my_debian
e70de35d058e   centos    "sleep 10000"   27 seconds ago   Up 25 seconds             my_centos
root@spinxaler-VirtualBox:~# docker exec -ti my_centos /bin/bash
[root@e70de35d058e /]# echo "My_centos_first_docker" > /data/centos_file
[root@e70de35d058e /]# exit
exit
root@spinxaler-VirtualBox:~# echo "My_host_locall" > /home/spinxaler/data/Host_file
root@spinxaler-VirtualBox:~# docker exec -ti my_debian /bin/bash
root@5c52b6a8e414:/# ls /data
Host_file  centos_file
root@5c52b6a8e414:/# exit
exit
root@spinxaler-VirtualBox:~# 
'''

## Обязательная задача 4

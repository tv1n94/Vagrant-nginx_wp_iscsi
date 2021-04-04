Данная конфигурация позволяет развернуть комплекс виртуальных машин на CentOS 7 в virtualbox через Vagrant: 
- 2 ВМ с nginx, php-fpm, wordpress с настроенным плавающим адресом keepalived. Между 2 ВМ настроен кластер Pacemaker с GFS2, для хранения статических данных. 
- 1 ВМ БД для WordPress
- 1 ВМ с ISCSI Target 

Настройка будет происходить через Ansible. 

Виртуальные машины будут развёрнуты с адресами из подсетей 192.168.2.0/24 и 192.168.3.0/24

Требования для запуска данной конфигурации: 
-Наличие Linux-машины
-Установленные пакеты Ansible 2.7 и Terraform.
-Установленная среда виртуализации Virtualbox и vagrant
-ПК не должен использовать сети 192.168.2.0/24 и 192.168.3.0/24

Описание файлов:
- Vagrantfile - файл с описанием конфигурации виртуальных машин, которые будут развернуты для дальнейшей настройки
- ansible.cfg - файл с настройками ansible
- hosts - файл с описанием реквизитов доступа в ВМ
- provision.yml - основной ansible-playbook. Разворачивает весь комплекс
- standby_node1.yml - файловер ноды node1
- unstandby_node1.yml - возвращение node1 в рабочее состояние

Конфигурационные файлы:
- keep_node1 - конфигурационный файл для Keepalived (Master)
- keep_node22 - конфигурационный файл для Keepalived (Backup)
- php.ini - конфигурационный файл для php-fpm
- wordpress_highload.conf - конфигурационный файл для nginx wordpress
- 50-server.cnf - конфигурационный файл mysql
- wp-config.php - конфигурационный файл для подключения к БД WordPress на другом сервере


Как развернуть конфигурацию: 
1) На подготовленную Linux-машину копируем все файлы из текущего репозитория
2) В файле hosts вносим актуальные пути до ssh-ключей Vagrant
3) Переходим в каталог Ansible: cd ansible
4) Запускаем команду: vagrant up && ansible-playbook -i hosts provision.yml

Проверка стенда: 

1) Подключаемся к node1, и проверяем работу кластера pacemaker:
alex@dell:~/Vagrant/nginx_wp_iscsi_db/ansible$ vagrant ssh node1
Last login: Sun Apr  4 22:52:19 2021 from 192.168.2.1
[vagrant@node1 ~]$ sudo pcs status
Cluster name: hacluster
Stack: corosync
Current DC: node2 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Sun Apr  4 22:57:27 2021
Last change: Sun Apr  4 22:52:21 2021 by root via cibadmin on node1

2 nodes configured
9 resource instances configured

Online: [ node1 node2 ]

Full list of resources:

 Clone Set: dlm-clone [dlm]
     Started: [ node1 node2 ]
 Clone Set: clvmd-clone [clvmd]
     Started: [ node1 node2 ]
 Clone Set: clusterfs-clone [clusterfs]
     Started: [ node1 node2 ]
 webserver      (ocf::heartbeat:nginx): Started node1
 php72-php-fpm  (systemd:php72-php-fpm.service):        Started node1
 keepalived     (systemd:keepalived):   Started node1

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
[vagrant@node1 ~]$ 

Далее заходим на веб-страницу: http://192.168.2.100:8080 и завершаем установку nginx

Проверить настройки nginx и php-fmp для Highload можно с помощью yandex-tank.
Ниже есть отдельный файл с инструкцией по проверке. На данный момент настройка выдерживает стабильно 2000 подключений. 

2) Тестово выключаем node1: ansible-playbook -i hosts standby_node1.yml

3) Возвращаем обратно node1 к работе: ansible-playbook -i hosts unstandby_node1.yml


Подключение к требуемой ВМ по ssh: vagrant ssh <имя ВМ>
Удаление стенда с ВМ: vagrant destroy -f

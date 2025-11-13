
## 1. Произведите базовую настройку устройств  
 ### ● Настройте имена устройств согласно топологии. Используйте полное доменное имя
    HQ-RTR | BR-RTR:  
    conf t
    hostname {hq-rtr.au-team.irpo, br-rtr.au-team.irpo}    
    HQ-SRV | HQ-CLI | BR-SRV:  
    hostnamectl set-hostname {hq-srv, hq-cli, br-srv}.au-team.irpo; exec bash  
 ### ● На всех устройствах необходимо сконфигурировать IPv4  
    Настройка адресов производится через nmtui  
 ### ● IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918   
    RFC 1918 включает себя следующие адреса:  
    10.0.0.0/8  
    172.16.0.0/12  
    192.168.0.0/16  
 ### ● Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 32 адресов  
    маска /27 255.255.255.224 - 32 адреса (30 используемых)
    192.168.0.0/27 192.168.0.1 – 192.168.0.30	192.168.0.31 - Broadcast  
 ### ● Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не менее 16 адресов  
    маска /27 255.255.255.224 - 32 адреса (30 используемых)
    192.168.0.32/27	192.168.0.33 – 192.168.0.62	192.168.0.63 - Broadcast  
 ### ● Локальная сеть в сторону BR-SRV должна вмещать не более 8 адресов  
    маска /29 255.255.255.248 - 8 адресов (6 используемых)
    192.168.0.64/29	192.168.0.65 – 192.168.0.70	192.168.0.71 - Broadcast  
    Сразу назначим адрес в сторону сервера
    Настройка производится на BR-RTR:
    en   
    conf t   
    port te1   
    service-instance toSRV   
    encapsulation untagged   
    end   
    wr mem      
    conf t   
    int SRV   
    ip add 192.168.0.65/29   
    connect port te1 service-instance toSRV   
    end   
    wr mem   
 ### ● Локальная сеть для управления(VLAN999) должна вмещать не более 16 адресов  
    маска /28 255.255.255.240 - 16 адресов (14 используемых)
    192.168.0.72/28	192.168.0.73 – 192.168.0.86	192.168.0.87 - Broadcast  
### ● Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 2, в качестве примера используйте Прил_3_О1_КОД 09.02.06-1-2026-М1
   ![Прил_3_О1_КОД 09.02.06-1-2026-М1](https://github.com/dizzamer/DEMO2026-Profile/blob/main/%D0%9F%D1%80%D0%B8%D0%BB_3_%D0%9E%D0%97_%D0%9A%D0%9E%D0%94%2009.02.06-1-2026-%D0%9C1.docx)
    
 | Имя Устройства   | IPv4                     |  Интерфейс  | NIC       | Шлюз         | 
 | ---------        | ---------                | ---------   | --------- | ---------    |
 | ISP              | NAT (inet)               | ens3        | Internet  |              |
 |                  | 172.16.1.14/28           | ens4        | ISP_HQ    |              |
 |                  | 172.16.2.14/28           | ens5        | ISP_BR    |              |
 | HQ-RTR           | 172.16.1.1/28            | te0         | ISP_HQ    | 172.16.1.14  |
 |                  | 192.168.0.73/28          | te1.999     | HQ_NET    |              |
 |                  | 192.168.0.1/27           | te1.100     | -         |              |
 |                  | 192.168.0.33/27          | te1.200     | -         |              |
 |                  | 172.16.0.1/30            | GRE         | TUN       |              |
 | HQ-SW            | 192.168.0.74/28          | ens3        | HQ_NET    |              |
 |                  | -                        | ens4        | SRV_NET   |              |
 |                  | -                        | ens5        | CLI_NET   |              |
 | HQ-SRV           | 192.168.0.2/27           | ens3        | SRV_NET   | 192.168.0.1  |
 | HQ-CLI           | 192.168.0.34/27(DHCP)    | ens3        | CLI_NET   | 192.168.0.33 |
 | BR-RTR           | 172.16.2.1/28            | te0         | ISP_BR    | 172.16.2.14  |
 |                  | 192.168.0.65/29          | te1         | BR_NET    |              |
 |                  | 172.16.0.2/30            | GRE         | TUN       |              |
 | BR-SRV           | 192.168.0.66/29          | ens3        | BR_NET    | 192.168.0.65 |

## 2. Настройте доступ к сети Интернет, на маршрутизаторе ISP:  
 ### ● Настройте адресацию на интерфейсах:  
    Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP  
    o Настройте маршруты по умолчанию там, где это необходимо   
    Маршруты по умолчанию настраиваются на роутерах:  
    HQ-RTR - ip route 0.0.0.0/0 172.16.1.14  
    BR-RTR - ip route 0.0.0.0/0 172.16.2.14  
    o Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.1.0/28  
    Настройка производится на HQ-RTR(EcoRouter):  
    en  
    conf t  
    port te0  
    service-instance toISP  
    encapsulation untagged  
    end  
    wr mem  
    en  
    conf t  
    int ISP  
    ip add 172.16.1.1/28  
    connect port te0 service-instance toISP  
    end  
    wr mem  
    o Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.2.0/28  
    Настройка производится на BR-RTR(EcoRouter):  
    en  
    conf t  
    port te0  
    service-instance toISP  
    encapsulation untagged  
    end  
    wr mem  
    en  
    conf t  
    int ISP  
    ip add 172.16.2.1/28  
    connect port te0 service-instance toISP  
    end  
    wr  mem  
    o На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR  
    для доступа к сети Интернет:  
    echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
    dnf install iptables-services –y   
    systemctl enable ––now iptables  
    iptables –t nat –A POSTROUTING –s 172.16.1.0/28 –o ens3 –j MASQUERADE  
    iptables –t nat –A POSTROUTING –s 172.16.2.0/28 –o ens3 –j MASQUERADE  
    iptables-save > /etc/sysconfig/iptables  
    systemctl restart iptables  
    nano /etc/sysconfig/iptables - не должно быть ничего лишнего - только настройка нашего ната, все остальное удаляем.    
    в случае если там есть то, что вы не добавляли - удалить, затем убрать из буфера старые правила с помощью iptables -F, далее перезагружаем службу iptables  
    iptables –L –t nat - должны высветится в Chain POSTROUTING две настроенные подсети.  
## 3. Создайте локальные учетные записи на серверах HQ-SRV и BR-SRV  
 ### ● Создание пользователя sshuser на серверах HQ-SRV | BR-SRV:    
    useradd -m -u 2026 sshuser  
    o Пароль пользователя sshuser с паролем P@ssw0rd  
    echo "sshuser:P@ssw0rd" | sudo chpasswd  
    o Идентификатор пользователя 2026  
    o Пользователь sshuser должен иметь возможность запускать sudo
    без дополнительной аутентификации.  
    usermod -aG wheel sshuser  
   ### ● Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BR-RTR  
    Настройка производится на EcoRouter HQ-RTR | BR-RTR:  
    conf t  
    username net_admin  
    o Пароль пользователя net_admin с паролем P@ssw0rd  
    password P@ssw0rd   
    o При настройке на EcoRouter пользователь net_admin должен обладать максимальными привилегиями  
    role admin  
    o При настройке ОС на базе Linux, запускать sudo без дополнительной аутентификации  
## 4. Настройте коммутацию в сегменте HQ следующим образом:  
 ### ● Предусмотреть возможность передачи трафика управления в VLAN 999    
    Настройка на HQ-RTR:  
    port te1  
    Service-instance toSW  
    Encapsulation dot1q 999  
    rewrite pop 1  
    end  
    wr mem  
    en  
    conf t  
    Int te1.999  
    ip add 192.168.0.73/28      
    connect port te1 service-instance toSW   
    end  
    wr mem  
    Настройка на HQ-SW:  
    Адресации так не должно быть  
    ovs-vsctl add-br hq-sw  
    ovs-vsctl add-port hq-sw ens3  
    ovs-vsctl set port ens3 vlan_mode=native-untagged tag=999 trunks=999,100,200  
    ovs-vsctl add-port hq-sw ovs0-vlan999 tag=999 -- set interface ovs0-vlan999 type=internal  
    ifconfig ovs0-vlan999 inet 192.168.0.74/28 up   
 ### ● Трафик HQ-SRV должен принадлежать VLAN 100    
    Настройка на HQ-RTR:  
    conf t  
    port te1   
    service-instance te1.100   
    encapsulation dot1q 100   
    rewrite pop 1   
    end   
    wr mem   
    int te1.100  
    ip add 192.168.0.1/27  
    connect port te1 service-instance te1.100  
    end  
    wr mem  
    Настройка на HQ-SW:  
    Адресации не должно быть
    Так как при настройке на HQ-SW бридж hq-sw уже создан, его создавать не нужно
    ovs-vsctl add-port hq-sw ens4  
    ovs-vsctl set port ens4 tag=100 trunks=100  
 ### ● Трафик HQ-CLI должен принадлежать VLAN 200    
    Настройка на HQ-RTR:  
    conf t  
    port te1  
    service-instance te1.200  
    encapsulation dot1q 200  
    rewrite pop 1  
    end  
    wr mem  
    int te1.200  
    ip add 192.168.0.33/28  
    connect port te1 service-instance te1.200 
    end  
    wr mem  
    Настройка на HQ-SW: 
    Адресации не должно быть
    Так как при настройке на HQ-SW бридж hq-sw уже создан, его создавать не нужно
    ovs-vsctl add-port hq-sw ens5  
    ovs-vsctl set port ens5 tag=200 trunks=200   
**● Сведения о настройке коммутации внесите в отчёт**  
## 5. Настройте безопасный удаленный доступ на серверах HQ-SRV и BR-SRV:  
 ### ● Для подключения используйте порт 2026  
    Перед настройкой выполните команду setenforce 0, далее проверяем командой: getenforce   
    должно быть состояние Permissive  
    dnf install openssh - если не установлен
    systemctl enable --now sshd
    echo Port 2026 >> /etc/ssh/sshd_config
  ### ● Разрешите подключения только пользователю sshuser  
    echo AllowUsers sshuser >> /etc/ssh/sshd_config
 ### ● Ограничьте количество попыток входа до двух  
    echo MaxAuthTries 2 >> /etc/ssh/sshd_config
 ### ● Настройте баннер «Authorized access only»  
    echo «Authorized access only» > /etc/ssh/sshd_banner
    echo Banner /etc/ssh/sshd_banner >> /etc/ssh/sshd_config
    systemctl restart sshd
## 6. Между офисами HQ и BR, на маршрутизаторах HQ-RTR и BR-RTR необходимо сконфигурировать ip туннель: 
  ### o На выбор технологии GRE или IP in IP - используем GRE
    Настройка на HQ-RTR:  
    Interface tunnel.1   
    Ip add 172.16.0.1/30   
    Ip mtu 1476   
    ip ospf network broadcast   
    ip ospf mtu-ignore   
    Ip tunnel 172.16.1.1 172.16.2.1 mode gre   
    end    
    wr mem    
    Настройка на BR-RTR:  
    Interface tunnel.1  
    Ip add 172.16.0.2/30  
    Ip mtu 1476  
    ip ospf mtu-ignore  
    ip ospf network broadcast  
    Ip tunnel 172.16.2.1 172.16.1.1 mode gre  
    end    
    ПРОВЕРЯЕМ ТУННЕЛЬ ПИНГОМ ОТ РОТУЕРА К РОУТЕРУ, БЕЗ ЭТОГО НЕ ПЕРЕХОДИМ К НАСТРОЙКЕ OSPF!!!
    Проверка на HQ-RTR: 
    ping 172.16.0.2  
    Проверка на BR-RTR:  
    ping 172.16.0.1
    Должно быть успешно с 2-х сторон!!!
  o Сведения о туннеле занесите в отчёт
## 7. Обеспечьте динамическую маршрутизацию на маршрутизаторах HQ-RTR и BR-RTR: сети одного офиса должны быть доступны из другого офиса и наоборот. Для обеспечения динамической маршрутизации используйте link state протокол на усмотрение участника: Будем использовать OSPF      
  ### ● Разрешите выбранный протокол только на интерфейсах в ip туннеле   
  ### ● Маршрутизаторы должны делиться маршрутами только друг с другом:   
    Настройка на HQ-RTR:  
    Conf t
    Router ospf 1
    Ospf router-id  172.16.0.1
    network 172.16.0.0 0.0.0.3 area 0
    network 192.168.0.0 0.0.0.31 area 0
    network 192.168.0.32 0.0.0.31 area 0
    passive-interface default
    no passive-interface tunnel.1
    Настройка на BR-RTR:  
    Conf t
    Router ospf 1
    Ospf router-id 172.16.0.2
    Network 172.16.0.0 0.0.0.3 area 0
    Network 192.168.0.64 0.0.0.7 area 0
    Passive-interface default
    no passive-interface tunnel.1  
  ### ● Обеспечьте защиту выбранного протокола посредством парольной защиты  
    Настройка производится на EcoRouter HQ-RTR:  
    router ospf 1  
    area 0 authentication  
    ex  
    interface tunnel.1   
    ip ospf authentication-key ecorouter  
    wr mem   
    Настройка производится на EcoRouter BR-RTR:  
    router ospf 1   
    area 0 authentication  
    ex   
    interface tunnel.1   
    ip ospf authentication-key ecorouter   
    wr mem   
  ● Сведения о настройке и защите протокола занесите в отчёт  
## 8. Настройка динамической трансляции адресов маршрутизаторах HQ-RTR и BR-RTR  
 ### ● Настройте динамическую трансляцию адресов для обоих офисов в сторону ISP.  
    Настройка производится на EcoRouter HQ-RTR: 
    ip nat pool nat1 192.168.0.0-192.168.0.31  
    ip nat source dynamic inside-to-outside pool nat1 overload interface ISP 
    ip nat pool nat2 192.168.0.32-192.168.0.63  
    ip nat source dynamic inside-to-outside pool nat2 overload interface ISP   
    Настройка производится на EcoRouter BR-RTR: 
    ip nat pool nat3 192.168.0.64-192.168.0.71  
    ip nat source dynamic inside-to-outside pool nat3 overload interface ISP 
### ● Все устройства в офисах должны иметь доступ к сети Интернет  
    Настройка производится на EcoRouter HQ-RTR:
    en
    conf t
    int ISP
    ip nat outside
    ex
    int te1.999
    ip nat inside
    ex
    int te1.100
    ip nat inside
    ex
    int te1.200
    ip nat inside
    Настройка производится на EcoRouter BR-RTR: 
    en
    conf t
    int ISP
    ip nat outside
    ex
    int SRV
    ip nat inside
    ex  
    Настройка производится на HQ-SRV:
    В nmtui прописывеем шлюз - 192.168.0.1  
    Настройка производится на BR-SRV:  
    В nmtui прописывет шлюз - 192.168.0.65  
## 9. Настройте протокол динамической конфигурации хостов для сети в сторону HQ-CLI:    
   ● Настройте нужную подсеть  
  ### ● Для офиса HQ в качестве сервера DHCP выступает маршрутизатор HQ-RTR    
    Настройка производится на EcoRouter HQ-RTR:  
    ip pool dhcpHQ 192.168.0.34-192.168.0.62  
    en  
    conf t  
    dhcp-server 1  
    mask 255.255.255.224  
    pool dhcpHQ 1  
    domain-name au-team.irpo  
    mask 255.255.255.224  
    dns 192.168.0.2     
    gateway 192.168.0.33     
    end  
    wr mem  
  ###  ● Клиентом является машина HQ-CLI.  
    interface te1.200
    dhcp-server 1
  ● Исключите из выдачи адрес маршрутизатора  
  ● Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR.  
  ● Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV.  
  ● DNS-суффикс для офисов HQ – au-team.irpo  
  ● Сведения о настройке протокола занесите в отчёт  
## 10. Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR:    
    ● Основной DNS-сервер реализован на HQ-SRV    
      dnf install bind -y  
      systemctl enable --now named  
      cp /etc/named.conf /etc/named.conf.backup - делаем бэкап файла  
      nano /etc/named.conf  
   ![named первая часть](https://github.com/dizzamer/DEMO2026-Profile/blob/main/namedconf1.png)  
   ![named вторая часть](https://github.com/dizzamer/DEMO2026-Profile/blob/main/namedconf3.png)  
      mkdir /var/named/master  
      nano /var/named/master/au-team  
   ![au team irpo зона](https://github.com/dizzamer/DEMO2026-Profile/blob/main/au-team1.png)  
      nano /var/named/master/168.192.zone  
      можно сделать через cp /var/named/master/au-team /var/named/master/168.192.zone, чтобы конфиг с нуля не писать  
   ![au team irpo зона](https://github.com/dizzamer/DEMO2026-Profile/blob/main/168.192zone.png)  
      chown -R root:named /var/named/master/  
      chown -R named:named /var/named  
      chown -R root:named /etc/named.conf    
      chmod 750 /var/named/   
      chmod 750 /var/named/master/    
      systemctl restart named   
      Проверить зоны можно командой named-checkconf -z    
    ![au team irpo зона](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chechkconf.png)  
      Для полной работоспособности на HQ-CLI нужно установить в качестве dns севрера HQ-SRV:  
      nano /etc/resolv.conf на всех устройствах должен иметь следюущий вид:  
    ![resolvconf](https://github.com/dizzamer/DEMO2026-Profile/blob/main/resolv.conf.png)  
      resolvectl dns ens3 192.168.0.2  
      Для полной работоспособности на HQ-RTR нужно установить в качестве dns севрера HQ-SRV:  
      ip name-server 192.168.0.2  
      На остальных устройствах делаем подобным образом.  
  ● Сервер должен обеспечивать разрешение имён в сетевые адреса устройств и обратно в соответствии с таблицей 3  
### Таблица 3. Таблица имен  
   | Устройство | Запись              | Тип    | 
   | ---------  |  ------             | ----   |
   | HQ-RTR     | hq-rtr.au-team.irpo | A,PTR  | 
   | BR-RTR     | br-rtr.au-team.irpo | A      |
   | HQ-SRV     | hq-srv.au-team.irpo | A,PTR  |
   | HQ-CLI     | hq-cli.au-team.irpo | A,PTR  |
   | BR-SRV     | br-srv.au-team.irpo | A      |
   | ISP (интерфейс направленный в сторону HQ-RTR)     | docker.au-team.irpo | A      | 
   | ISP (интерфейс направленный в сторону BR-RTR)     | web.au-team.irpo    | A      |  
      
  ● В качестве DNS сервера пересылки используйте любой общедоступный DNS сервер  
## 11. Настройте часовой пояс на всех устройствах, согласно месту проведения экзамена.  
   ### Настройка проивзодится на всех устройствах:  
    timedatectl set-timezone Europe/Moscow  
## Необходимые приложения для модуля 1:  
   Прил_1_О1_КОД 09.02.06-1-2026-М1: Шаблон отчета  
   Прил_3_О1_КОД 09.02.06-1-2026-М1: Пример заполнения таблицы адресов  
   Прил_4_О1_КОД 09.02.06-1-2026-М1: Инструкции по оформлению отчёта  

# Модуль № 2:Организация сетевого администрирования операционных систем  

## 1.	Настройте доменный контроллер Samba на машине BR-SRV.   
  ### Настройка проивзодится на BR-SRV:  
   ## Предварительная настройка сервера:  
     выставляем 192.168.0.2 в качестве нащего днс сервера на линке в nmtui и домен поиска au-team.irpo  
     setenforce 0  
     nano /etc/selinux  
     Замените в файле конфигурации /etc/selinux/config режим enforcing на permissive   
     dnf install samba* krb5* -y  
   ## Создание домена под управлением контроллера домена Samba DC:  
     nano /etc/krb5.conf
     # Меняем следующие строки
     default_realm = au-team.irpo
     [realms]
     au-team.irpo = {
     kdc = dc1.au-team.irpo
     admin_server = dc1.au-team.irpo
     }
     [domain_realm]
     .au-team.irpo = AU-TEAM.IRPO
     au-team.irpo = AU-TEAM.IRPO
   ## Конфигурирование сервера с помощью утилиты samba-tool  
      Файла /etc/samba/smb.conf быть не должно, он сам создаст.
      rm -rf /etc/samba/smb.conf
      samba-tool domain provision --use-rfc2307 --interactive
      1.REALM [AU-TEAM.IRPO]:
      2.DOMAIN [AU-TEAM]:
      3.Server Role: DC
      4.DNS backend: BIND9_DLZ
      Запустите и добавьте в автозагрузку службы samba:  
      systemctl status samba  
   ## •	Создайте 5 пользователей для офиса HQ: имена пользователей формата user№.hq. Создайте группу hq, введите в эту группу созданных пользователей  
     sudo samba-tool group add hq  
     for i in {1..5}; do  
     sudo samba-tool user create user$i.hq P@ssw0rd$i  
     sudo samba-tool group addmembers hq user$i.hq  
     done    
   ## •	Введите в домен машину HQ-CLI  
      Ввод в домен HQ-CLI
      Переводим DHCP в полуавтоматический режим и указываем собственный DNS
   ![fstab](https://github.com/dizzamer/DEMO2026-Profile/blob/main/semiautodhcp.png) 
   ![fstab](https://github.com/dizzamer/DEMO2026-Profile/blob/main/domain.png)
   ![fstab](https://github.com/dizzamer/DEMO2026-Profile/blob/main/success.png)
   ## 2.	Сконфигурируйте файловое хранилище:  
 ### Настройка проивзодится на HQ-SRV:
  Перед тем как начать проверяем, что установлены следюущие пакеты
  dnf install mdadm nfs-utils -y
## •	При помощи трёх дополнительных дисков, размером 1Гб каждый, на HQ-SRV сконфигурируйте дисковый массив уровня 5  
    mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd     
 ![mdadmcreate](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mdadm_create.png)    
 ![mdaddetail](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mdadm_detail.png)  
## •	Имя устройства – md0, конфигурация массива размещается в файле /etc/mdadm.conf  
    mdadm --detail --scan >> /etc/mdadm.conf      
 ## •	Обеспечьте автоматическое монтирование в папку /raid     
    Добавляем в /etc/fstab:    
    nano /etc/fstab  
    /dev/md0 /raid ext4 defaults 0 0  
![fstab](https://github.com/dizzamer/DEMO2026-Profile/blob/main/etcfstab.png)  
 ## •	Создайте раздел, отформатируйте раздел, в качестве файловой системы используйте ext4  
     mkfs.ext4 /dev/md0  
   ![mkfs](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mkfs.png)   
 ## •	Создаем точку монтирования и примонтируемся     
    mkdir -p /raid   
    mount -a   
  ![mount](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mount1.png)   
 ## 3. Настройте сервер сетевой файловой системы(nfs) на HQ-SRV
 ### Настройка проивзодится на HQ-SRV:
  ## • в качестве папки общего доступа выберите /raid/nfs, доступ для чтения и записи для всей сети в сторону HQ-CLI   
  ## •	Создаем папку для NFS  
    mkdir -p /raid/nfs  
    chmod 777 /raid/nfs  
 ![mkdir_nfs](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mkdir_nfsn.png)  
 ## Настройка экспорта  
    Добавляем в /etc/exports:  
    nano /etc/exports  
    /raid/nfs 192.168.0.32/27(rw,sync,no_root_squash,no_all_squash,subtree_check)
![exports](https://github.com/dizzamer/DEMO2026-Profile/blob/main/exports.png)  
  ## Применяем изменения и перезагружаем службу
    exportfs -rav  
    systemctl restart nfs-server  
 ### Настройка проивзодится на HQ-CLI:
  ## •	На HQ-CLI настройте автомонтирование в папку /mnt/nfs  
      Добавляем в /etc/fstab:    
      nano /etc/fstab  
      hq-srv:/raid/nfs /mnt/nfs nfs defaults 0 0
  ![fstab_hqcli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/etcfstab%20cli.png)  
  ## Создаем точку монтирования и примонтируемся  
    mkdir -p /mnt/nfs  
    mount -a 
  ![mountdir_hqcli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/nount_cli.png)  
  ## Проверка монтирования
      После этого при создании файла на клиенте, он должен появляться и на сервере
   •	Основные параметры сервера отметьте в отчёте  
  ## 4.	Настройте службу сетевого времени на базе сервиса chrony на маршрутизаторе ISP  
  ### Настройка проивзодится на ISP:     
     •	Стратум сервера - 5   
     В РЕД ОС сервис Chrony уже установлен по умолчанию.  
     Вносим изменения в файл конфигурации:    
     Добавляем сети, которые необходимы и выставляем stratum 5   
     nano /etc/chrony.conf    
  ![chrony_conf](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chrony_conf.png)  
     Переводим службу в автозапуск и запускаем:   
     systemctl enable --now chronyd   
  ## •	В качестве клиентов ntp настройте: HQ-SRV, HQ-CLI, BR-RTR, BR-SRV.   
  ### Настройка проивзодится на HQ-RTR:  
     en  
     conf t  
     ntp server 172.16.1.14  
     ex  
     wr mem  
  ### Настройка проивзодится на HQ-CLI:  
     В РЕД ОС сервис Chrony уже установлен по умолчанию. 
     Вносим изменения в файл конфигурации:  
     Комментируем/удаляем строки, где указаны ntp сервера и добавляем наш сервер 172.16.1.14  
     nano /etc/chrony.conf  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chrony_conf_cli.png)
    Переводим службу в автозапуск и запускаем:    
    systemctl enable --now chronyd  
    Проевряем настройку командой chronyc sources -v, должен отобразиться наш сервер:  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chronyc_sources_cli.png)  
    Не паникуем, может сразу не появится, рестартим сервис несколько раз командой systemctl restart chronyd  
  ### Настройка проивзодится на HQ-SRV:  
    Вносим изменения в файл конфигурации:  
    Комментируем/удаляем строки, где указаны ntp сервера и добавляем наш сервер 172.16.1.14   
    nano /etc/chrony.conf  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chronyc_conf_hqsrv.png)  
    Переводим службу в автозапуск и запускаем:    
    systemctl enable --now chronyd  
    Проевряем настройку командой chronyc sources, должен отобразиться наш сервер:  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chronyc_sources_hqsrv.png)  
    Не паникуем, может сразу не появится рестартим сервис несколько раз командой systemctl restart chronyd  
  ### Настройка проивзодится на BR-RTR:  
    en  
    conf t  
    ntp server 172.16.2.14  
    ex  
    wr mem  
  ### Настройка производится на BR-SRV:   
    Вносим изменения в файл конфигурации:  
    Комментируем/удаляем строки, где указаны ntp сервера и добавляем наш сервер 172.16.2.14   
    nano /etc/chrony.conf  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chronyc_conf_brsrv.png)  
    Переводим службу в автозапуск и запускаем:    
    systemctl enable --now chronyd  
    Проевряем настройку командой chronyc sources, должен отобразиться наш сервер:  
  ![chrony_conf_cli](https://github.com/dizzamer/DEMO2026-Profile/blob/main/chronyc_sources_brsrv.png)  
    Не паникуем, может сразу не появится рестартим сервис несколько раз командой systemctl restart chronyd  
## 5.	Сконфигурируйте ansible на сервере BR-SRV  
  ### Настройка подключения по ssh BR-RTR | HQ-RTR
     Настройка производится на HQ-RTR:  
     en  
     conf  
     security-profile 1  
     rule 1 permit tcp any eq 22 any  
     end  
     wr mem  
     configure
     ip vrf vrf0  
     transport input ssh    
     security 1 vrf vrf0  
     end  
     wr mem  
     conf
     no security default  
     Настройка производится на BR-RTR: 
     conf  
     security-profile 1  
     rule 1 permit tcp any eq 22 any  
     end  
     wr mem  
     configure
     ip vrf vrf0  
     transport input ssh    
     security 1 vrf vrf0  
     end  
     wr mem  
     conf
     no security default
  ## •	Сформируйте файл инвентаря, в инвентарь должны входить HQ-SRV, HQ-CLI, HQ-RTR и BR-RTR  
   ### Настройка производится на HQ-SRV:
       sudo dnf install ansible sshpass -y 
   ### Настройка производится на BR-SRV:  
    • Рабочий каталог ansible должен располагаться в /etc/ansible  
      dnf install ansible sshpass -y  
      1) В файле можно прописывать как ip адреса так и имена хостов, сделаем следующим образом.  
      Так как у нас порт для покдлючения к серверам 2026 и клиентам 22, укажим необходимые переменные для подключения  
      Для роутера так же указываем переменные, для подключения к роутерам будем использовать пользователя net_admin
      nano /etc/ansible/inventory.ini  
      [clients]
      hq-cli ansible_host=192.168.0.34
        
      [servers]
      hq-srv ansible_host=192.168.0.2
         
      [routers]
      hq-rtr ansible_host=172.16.1.1
      br-rtr ansible_host=172.16.2.1
         
      [clients:vars]
      ansible_port=22
      ansible_user=student
      ansible_password=student
         
      [servers:vars]
      ansible_port=2026 
      ansible_user=sshuser 
      ansible_password=P@ssw0rd 
 
      [routers:vars]  
      ansible_port=22  
      ansible_user=net_admin    
      ansible_password=P@ssw0rd  
      ansible_connection=network_cli  
      ansible_network_os=ios  
   ![inventory](https://github.com/dizzamer/DEMO2026-Profile/blob/main/inventory_ini1.png)  
  ### •	Все указанные машины должны без предупреждений и ошибок отвечать pong на команду ping в ansible посланную с BR-SRV  
    Пингуем удаленные хосты с помощью Ansible находясь в пользователе sshuser:  
    ansible -i /etc/ansible/inventory.ini all -m ping  
    В результате под каждым хостом должно быть написано "ping": "pong".    
   ![inventory](https://github.com/dizzamer/DEMO2026-Profile/blob/main/ansible_ping.png)   
## 6.	Развертывание приложений в Docker на сервере BR-SRV. 
    Установка необходимых пакетов:  
    dnf install docker-ce docker-ce-cli docker-compose -y  
    systemctl enable docker --now
    Добавляем текущего пользователя в группу докер, текущий пользователь - student   
    usermod -aG docker $USER  
###  • Средствами docker должен создаваться стек контейнеров с веб приложением и базой данных
     • Используйте образы site_latest и mariadb_latest располагающиеся в
     директории docker в образе Additional.iso 
   ![wikiyml](https://github.com/dizzamer/DEMO2026-Profile/blob/main/mntdocker.png) 
     • Основной контейнер testapp должен называться testapp
     • Контейнер с базой данных должен называться db
 ###  • Импортируйте образы в docker, укажите в yaml файле параметры подключения к СУБД, имя БД - testdb, пользователь testс паролем P@ssw0rd, порт приложения 8080, при необходимости другие параметры
      docker load < /mnt/docker/docker/mariadb_latest.tar
      docker load < /mnt/docker/docker/site_latest.tar
      Для написания web.yaml в качестве подсказки можно использовать файл readmetxt, который лежит в месте образов:   
   ![readmetxt](https://github.com/dizzamer/DEMO2026-Profile/blob/main/readmetxt.png)
   ### Готовим наш yaml файл  
      nano web.yaml
       services:
    web:
      container_name: testapp
      image: site:latest
      restart: always 
      ports: 
        - 8080:8000 
      networks: 
        - testapp-net 
      environment: 
        DB_HOST: db 
        DB_PORT: "3306" 
        DB_NAME: testdb 
        DB_USER: testc 
        DB_PASS: P@ssw0rd 
        DB_TYPE: maria 
      depends_on: 
        - db 
    db: 
      container_name: testdb 
      image: mariadb:10.11 
      restart: always 
      ports: 
        - "3306:3306" 
      environment: 
        MARIADB_ROOT_PASSWORD: Passw0rd  
        MARIADB_DATABASE: testdb  
        MARIADB_USER: testc  
        MARIADB_PASSWORD: P@ssw0rd 
      networks: 
        - testapp-net     
   Конфигурационный файл:    
 ![webyaml](https://github.com/dizzamer/DEMO2026-Profile/blob/main/webyaml.png) 
   ### Поднимаем стек контейнеров с помощью команды:  
       docker compose -f web.yml up -d   
     • Приложение должно быть доступно для внешних подключений через порт 8080   
 ![web](https://github.com/dizzamer/DEMO2026-Profile/blob/main/compose_web.png)  
## 7.	Разверните веб приложениена сервере HQ-SRV:   
### Подготовка   
    Переводим selinux в состояние Permissive:  
    setenforce 0    
    Проверяем:  
    getenforce  
    Должно быть состояние: Permissive  
    Далее устанавливаем необходимые пакеты:  
    dnf install httpd mariadb-server mariadb php php-cli php-common php-fpm php-gd php-intl php-json php-mysqlnd php-pdo php-xml php-xmlrpc php-soap -y    
## •	Используйте веб-сервер apache  
    systemctl enable --now httpd   
    Создаем конфигурационный файл:   
    nano /etc/httpd/conf.d/web.conf   
    <VirtualHost *:80>   
        DocumentRoot "/var/www/html"   
        ServerName hq-srv  
        <Directory "/var/www/html/">   
            AllowOverride All   
            Require all granted   
        </Directory>   
    </VirtualHost>  
   ![dumpsql](https://github.com/dizzamer/DEMO2026-Profile/blob/main/webconf.png)    
## •	В качестве системы управления базами данных используйте mariadb   
     systemctl enable --now mariadb  
     mysql_secure_installation  
     Там везде вводим y, задаем пароль для пользователя root - P@ssw0rd  
## •	Файлы веб приложения и дамп базы данных находятся в директории web образа Additional.iso  
![dumpsql](https://github.com/dizzamer/DEMO2026-Profile/blob/main/filewebapp.png)  
## •	Выполните импорт схемы и данных из файла dump.sql в базу данных webdb  
     Для начала создадим базу данных:  
     mariadb -u root -p  
     CREATE DATABASE webdb;
     •	Создайте пользователя webс паролем P@ssw0rd и предоставьте ему права доступа к этой базе данных   
     CREATE USER 'webc'@'localhost' IDENTIFIED BY 'P@ssw0rd';   
     GRANT ALL PRIVILEGES ON webdb.* TO 'webc'@'localhost';   
     FLUSH PRIVILEGES;    
     EXIT;   
     Делаем импорт базы данных:     
     mariadb -u root -p webdb < ./dump.sql  
## •	Файлы index.php и директорию images скопируйте в каталог веб сервера apache  
   ![dumpsql](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cpvarwww.png) 
## •	В файле index.php укажите правильные учётные данные для подключения к БД  
   ![dumpsql](https://github.com/dizzamer/DEMO2026-Profile/blob/main/indexphp.png) 
## •	Запустите веб сервер и убедитесь в работоспособности приложения  
     Перезапускаем веб-сервер:  
     systemctl restart httpd  
   ![dumpsql](https://github.com/dizzamer/DEMO2026-Profile/blob/main/webapp.png) 
## •	Основные параметры отметьте в отчёте  
•	На главной странице должен отражаться номер рабочего места в виде арабской цифры, других подписей делать не надо  
•	Основные параметры отметьте в отчёте  
## 8.	На маршрутизаторах сконфигурируйте статическую трансляцию портов  
### •	Пробросьте порт 8080 в порт приложения testapp BR-SRV на маршрутизаторе BR-RTR, для обеспечения работы приложения testapp извне  
     Настройка производится на EcoRouter BR-RTR:  
     ip nat source static tcp 192.168.0.66 8080 172.16.2.1 8080
### •	Пробросьте порт 8080 в порт веб приложения на HQ-SRV на маршрутизаторе HQ-RTR, для обеспечения работы веб приложения извне  
     Настройка производится на EcoRouter HQ-RTR:  
     ip nat source static tcp 192.168.0.2 80 172.16.1.1 8080
### •	Пробросьте порт 2026 на маршрутизаторе HQ-RTR в порт 2026 сервера HQ-SRV, для подключения к серверу по протоколу ssh из внешних сетей  
     Настройка производится на EcoRouter HQ-RTR:  
     ip nat source static tcp 192.168.0.2 2026 172.16.1.1 2026  
### • Пробросьте порт 2026 на маршрутизаторе BR-RTR в порт 2026 сервера BR-SRV, для подключения к серверу по протоколу ssh из внешних сетей.
      Настройка производится на EcoRouter BR-RTR:  
     ip nat source static tcp 192.168.0.66 2026 172.16.2.1 2026  
## 9.	Настройте веб-сервер nginx как обратный прокси-сервер на ISP  
     Переводим selinux в состояние Permissive:  
     setenforce 0    
     Проверяем:  
     getenforce  
     Должно быть состояние: Permissive
     dnf install nginx -y  
     systemctl enable --now nginx  
     Перед настройкой конфига укажем значение server_names_hash_bucket_size 64
   ![bucket_hash](https://github.com/dizzamer/DEMO2026-Profile/blob/main/nginx0.png)
  ### •	При обращении по доменному имени web.au-team.irpo у клиента должно открываться веб приложение на HQ-SRV 
      nano /etc/nginx/nginx.conf  
      server {
        server_name web.au-team.irpo; 
        location / { 
            proxy_pass http://172.16.1.14:8080; 
            proxy_redirect     off; 
            proxy_set_header   Host             $host; 
            proxy_set_header   X-Real-IP        $remote_addr; 
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
            auth_basic "Restricted area"; 
            auth_basic_user_file /etc/nginx/.htpasswd;  
        } 
     }
![nginx1](https://github.com/dizzamer/DEMO2026-Profile/blob/main/nginx3web1.png)
  ### • При обращении по доменному имени docker.au-team.irpo клиента должно открываться веб приложение testapp
       nano /etc/nginx/nginx.conf
       server { 
        server_name docker.au-team.irpo; 
        location / { 
            proxy_pass http://172.16.2.14:8080; 
            proxy_redirect     off; 
            proxy_set_header   Host             $host; 
            proxy_set_header   X-Real-IP        $remote_addr; 
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for; 
        }
      }
![nginx2](https://github.com/dizzamer/DEMO2026-Profile/blob/main/nginx2docker1.png)
## 10.	На маршрутизаторе ISP настройте web-based аутентификацию:  
  ### •	При обращении к сайту web.au-team.irpo клиенту должно быть предложено ввести аутентификационные данные    
      • В качестве логина для аутентификации выберите WEBс паролем P@ssw0rd
      • Выберите файл /etc/nginx/.htpasswd в качестве хранилища учётных записей  
      dnf install httpd-tools -y
      htpasswd -c /etc/nginx/.htpasswd WEBc  
      Вводим P@ssw0rd
      • При успешной аутентификации клиент должен перейти на веб сайт.  
## 11.	Удобным способом установите приложение Яндекс Браузер для организаций на HQ-CLI  
•	Установку браузера отметьте в отчёте  

# Модуль № 3:Эксплуатация объектов сетевой инфраструктуры    

## 1. Выполните импорт пользователей в домен au-team.irpo:  
### • В качестве файла источника выберите файл users.csv располагающийся в образе Additional.iso  
    mkdir -p /mnt/samba  
    mount /dev/sr1 /mnt/samba  
### • Пользователи должны быть импортированы со своими паролями и другими атрибутами   
    Настройка производится на серверве BR-SRV:   
    nano import.sh   
    #!/bin/bash   
    tail -n +2 /mnt/samba/Users.csv | while IFS=';' read -r firstName lastName _ _ ou _ _ _ _ password    
    do  
    samba-tool ou create "OU=$ou"  
    samba-tool user create "${firstName}${lastName}" "P@ssw0rd1" \   
    --userou="OU=$ou"    
    done  
    chmod +x import/sh  
    ./import.sh  
![bash](https://github.com/dizzamer/DEMO2026-Profile/blob/main/importsh.png)    
### • Убедитесь, что импортированные пользователи могут войти на машину HQ-CLI  
Вход выполнен для пользователя выполнен  
![sambauser](https://github.com/dizzamer/DEMO2026-Profile/blob/main/user_samba1.png)
## 2. Выполните настройку центра сертификации на базе HQ-SRV:
### • Необходимо использовать отечественные алгоритмы шифрования
### • Сертификаты выдаются на 30дней
### • Обеспечьте доверие сертификату для HQ-CLI
### • Выдайте сертификаты веб серверам
### • Перенастройте ранее настроенный реверсивный прокси nginx на
протокол https
### • При обращении к веб серверам https://web.au-team.irpo и
https://docker.au-team.irpo у браузера клиента не должно возникать
предупреждений.
## 3. Перенастройте ip-туннель с базового до уровня туннеля, обеспечивающего шифрование трафика
### • Настройте защищенный туннель между HQ-RTR и BR-RTR  
### • Внесите необходимые изменения в конфигурацию динамической маршрутизации, протокол динамической маршрутизации должен возобновить работу после перенастройки туннеля  
### • Выбранное программное обеспечение, обоснование его выбора и его  
основные параметры, изменения в конфигурации динамической  
маршрутизации отметьте в отчёте.  
## 4. Настройте межсетевой экран на маршрутизаторах HQ-RTR и BR-RTR на сеть в сторону ISP
### • Обеспечьте работу протоколов http, https, dns, ntp, icmp или
дополнительных нужных протоколов
### • Запретите остальные подключения из сети Интернет во внутреннюю
сеть.
72
## 5. Настройте принт-сервер cups на сервере HQ-SRV:  
### • Опубликуйте виртуальный pdf-принтер   
   dnf isntall cups -y   
   systemctl enable --now cupsd cupspdf  
   делаем бэкап файла cp /etc/cups/cupsd.conf /etc/cups/cupsd.conf.backup  
   nano /etc/cups/cupsd.conf   
   Нужно изменить строку Listen localhost:631 на Listen *:631     
   В строке Restrict access to server... добавить Allow all    
   <Location />  
     Order allow, deny  
     Allow all  
   <Location>  
   В строке Restrict access to admin pages... добавить Allow all   
   <Location />  
     Order allow, deny  
     Allow all  
   <Location>   
 ![cupsconf](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupsdconf.png)   
  systemctl restart cupsd   

### • На клиенте HQ-CLI подключите виртуальный принтер как принтер по умолчанию.  
    Настройка производится на HQ-CLI:  
    Переводим SELinux в состояние Permissive  
    setenforce 0  
    getenforce  
  Далее заходим по админской учеткой student:student, должен появится принтер  
  ![cupsadminlog](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupscheckpr.png)  

  ![cupsadminlog](https://github.com/dizzamer/DEMO2026-Profile/blob/main/nlockprinter.png)  
  
  ![cupsadminlog](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupspdfadd3.png)    

  ![cupsadd4](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupspdfadd4.png)  

  ![cupsadd5](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupspdfadd5.png)  

  ![cupsadd5](https://github.com/dizzamer/DEMO2026-Profile/blob/main/cupspdfadd6.png)  
## 6. Реализуйте логирование при помощи rsyslog на устройствах HQ-RTR, BR-RTR, BR-SRV:
### • Сервер сбора логов расположен на HQ-SRV, убедитесь, что сервер не
является клиентом самому себе
### • Приоритет сообщений должен быть не ниже warning
### • Все журналы должны находиться в директории /opt. Для каждого
устройства должна выделяться своя поддиректория, которая совпадает с
именем машины
### • Реализуйте ротацию собранных логов на сервере HQ-SRV:
### • Ротируются все логи, находящиеся в директории и
поддиректориях /opt
### • Ротация производится один раз в неделю
### • Логи необходимо сжимать
### • Минимальный размер логов для ротации – 10МБ.
## 7. Насервере HQ-SRV реализуйте мониторинг устройств с помощью открытого программного обеспечения
### • Обеспечьте доступность по URL - http://mon.au-team.irpo для сетей
офиса HQ, внесите изменения в инфраструктуру разрешения доменных
имён
### • Мониторить нужно устройства HQ-SRV и BR-SRV
### • В мониторинге должны визуально отображаться нагрузка на ЦП, объем
занятой ОП и основного накопителя
### • Логин и пароль для службы мониторинга admin P@ssw0rd
### • Организуйте доступ к мониторингу для HQ-CLI, без внешнего доступа
73
### • Выбор программного обеспечения, основание выбора и основные
параметры с указанием порта, на котором работает мониторинг,
отметьте в отчёте
## 8. Реализуйте механизм инвентаризации машин HQ-SRV и HQ-CLI через Ansible на BR-SRV:
### • Плейбук должен собирать информацию о рабочих местах:
### • Имя компьютера
### • IP-адрес компьютера
### • Плейбук, должен быть размещен в директории /etc/ansible, отчёты в
поддиректории PC-INFO, в формате .yml. Файлы должны называется
именем компьютера, который был инвентаризирован
### • Файл плейбука располагается в образе Additional.iso в директории
playbook
## 9. На HQ-SRV настройте программное обеспечение fail2ban для защиты ssh  
      Настройка производится на HQ-SRV:  
      dnf install fail2ban  
      systemctl enable --now fail2ban  
      Все конфигурационные файлы программы находятся в каталоге /etc/fail2ban. Для установки собственных настроек необходимо создать файл с таким же именем и расширением .local.  
### • Укажите порт ssh  
### • При 3 неуспешных авторизациях адрес атакующего попадает в бан  
### • Бан производится на 1минуту  
      Сначала копируем содержимое jail.conf в jail.local:  
      cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local  
      nano /etc/fail2ban/jail.local  
      [sshd]  
      enabled  = true  
      filter   = sshd  
      action   = iptables[name=SSH, port=ssh, protocol=tcp]  
      logpath  = /var/log/auth.log  
      findtime    = 600  
      maxretry    = 3  
      bantime     = 60  
      systemctl restart fail2ban  
      Можно проверить введя 3 раза неправильно пароль при подключении к ssh,  
      после этого пробуем подключиться еще раз, должно быть Connection refused  
## 10.Настройка резервного копирования директории сервера HQ-SRV:
### • На HQ-SRV развернуть программное обеспечение для резервного
копирования и восстановления данных с защитой от вирусов-
шифровальщиков
### • В качестве решения рекомендуется использовать программное
обеспечение Кибер Бэкап версии 17.4 или аналог
### • Настройте организацию irpo
### • Настройте пользователя с правами администратора на сервере
HQ-SRV, имя пользователя irpoadmin с паролем P@ssw0rd
### • Установите на HQ-CLI агент с функциями узла хранилища и
подключите его к серверу управления
74
### • На узле хранилища HQ-CLI создайте директорию /backup и
выберите её в качестве устройства хранения
### • Создайте два плана резервного копирования для сервера HQ-SRV
### • план для резервного копирования директории /etc и всех её
поддиректорий
### • план для резервного копирования базы данных webdb типа
mysql
### • Выполните резервное копирование директории /etc и всех её
поддиректорий сервера HQ-SRV на узел хранения HQ-CLI
###  • Выполните резервное копирование базы данных webdb сервера
HQ-SRV на узел хранения HQ-CLI


 

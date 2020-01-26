# Настройка IPSec over L2TP 
Server - Ubuntu 16.04

Client - Ubuntu 18.04

## Настройка сервера L2TP 

### Шаг 1. Установка StrongSwan и xl2tpd

StrongSwan является демоном IPSEC, который мы настроим как наш VPN-сервер. Обновим пакеты и установим нужные компоненты:

```
apt-get update
apt-get install strongswan xl2tpd
```
    
### Шаг 2. Конфигурация ipsec.conf

Внесем изменения в конфигурационный файл nano /etc/ipsec.conf
```
conn L2TP-IPSEC
    authby=secret
    rekey=no
    keyingtries=3
    type=transport
    esp=aes128-sha1
    ike=aes128-sha-modp1024
    ikelifetime=8h
    keylife=1h
    left= # your router's external IP 
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
    rightsubnet=0.0.0.0/0
    auto=add
    dpddelay=30
    dpdtimeout=120
    dpdaction=clear
    #force all to be nat'ed. because of iOS
    forceencaps=yes
```

В строчке left вставляем наш внешний ip адрес.

### Шаг 3. Настройка Pre-Shared Key

Настроим Pre-Shared Key, чтобы можно было подключиться к нашему VPN. Заходим в nano /etc/ipsec.secrets

```
ip_address %any : PSK "your_psk"
```

Вместо ip_address вставляем внешний ip адрес сервера.
В кавычках прописываем любой Pre-Shared Key, например, "private".

### Шаг 4. Настройка L2TP

Внесём изменения в конфигурационный файл nano /etc/ppp/options.xl2tpd

```
require-mschap-v2
refuse-mschap
ms-dns 8.8.8.8
ms-dns 8.8.4.4
asyncmap 0
auth
crtscts
idle 1800
mtu 1410
mru 1410
connect-delay 5000
lock
hide-password
local
#debug
modem
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
```

Сохраняемся и выходим из файла. Далее открываем файл nano /etc/xl2tpd/xl2tpd.conf 

Общие настройки находятся в секции global:

```
[global]
ipsec saref = no
debug tunnel = no
debug avp = no
debug network = no
debug state = no
access control = no
rand source = dev
port = 1701
auth file = /etc/ppp/chap-secrets
```

Настройки сервера находятся в секции lns:

```
[lns default]
ip range = 192.168.1.10-192.168.122.20
local ip = 192.168.1.1
require authentication = yes
name = l2tp
pass peer = yes
ppp debug = no
length bit = yes
refuse pap = yes
refuse chap = yes
pppoptfile = /etc/ppp/options.xl2tpd
```

Сохраняем и закрываем файл.


### Шаг 5. Настройка данных пользователя

В файле /etc/ppp/chap-secrets указываются аутентификационные данные пользователей для CHAP аутентификации:

```
test    l2tpd     TestTest      "*"
```

test - наш логин

l2tpd - имя сервера, к которому можно подсоединиться (его можно заменить на *)

TestTest - наш пароль

"*" - Разрешает соединения с любых IP

### Шаг 6. Запуск сервера

Запускаем сервер командой:

```
service xl2tpd start
```

Наш VPN IPSec over L2TP сервер настроен.

## Настройка клиента L2TP 

### Шаг 1. Установка 

Обновляем и устанавливаем следующие зависимости:

```
sudo apt-get update
sudo apt-get install network-manager-l2tp
sudo apt-get install network-manager-l2tp-gnome
```

### Шаг 2. Подключение

Кликаем в верхнем правом углу на значок "network" и затем нажимаем на иконку настроек. 
Нажимаем "+" добавить VPN и выбираем L2TP.
В поле "name" пишем любое название нашего подключения. 
В "gateway" вставляем адрес нашего сервера.
В "user" и "password" пишем те данные, которые прописывали в файле chap-secrets сервера.
Затем жмем на кнопку "IPsec Settings". Ставим галочку "Enable IPsec tunnel to L2TP host" и в поле "Pre-Shared Key" пишем ключ, который мы задавали в файле ipsec.secrets сервера. Нажимаем везде "ok" и включаем VPN соединение. Клиент подключен.












### Полезные вики:
https://openwrt.org/ - вики по OpenWrt, полезные гайды по настройке служб
https://sing-box.sagernet.org/configuration/ - вики по ядру sing-box

### Прошивка роутера

 Для работы настройки VPN на роутере под управлением OpenWrt, необходимо его предварительно прошить. Проверить поддержку устройства можно на официальном сайте OpenWrt 
	https://toh.openwrt.org/ 
После прошивки роутера и проверки его работоспособности, можно подключаться к системе по SSH, пользователь по умолчанию root, пароль пустой 
### Установка и настройка sing-box
Данный гайд протестирован на версиях OpenWrt `25.12.3,24.10.4` на устройствах ASUS RT-AX1800U, Xiaomi Redmi AX6000
Для установки пакетов в OpenWrt используется пакетный менеджер `opkg` или `apk`, в зависимости от версии прошивки. В инструкции будем использовать `opkg`

#### 1. Установка sing-box 
`opkg update && opkg install sing-box`

#### 2. Настройка TUN интерфейса в /etc/config/network/
Добавить в конфиг настроек сети строки, сам интерфейс создавать не нужно, поскольку его создаст ядро sing-box
	
	config interface 'proxy'
        option proto 'none'
        option device 'tun0'
        option defaultroute '0'
        option delegate '0'
        option auto '1'

#### 3. Настройка Firewalld в /etc/config/firewall/
Добавить в конфиг строки, разрешение на редирект с LAN в TUN

	config zone
        option name 'proxy'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        option masq '1'
        list network 'proxy'

	config forwarding
        option src 'lan'
        option dest 'proxy'

#### 4. Перезапуск служб 
	 service network restart

#### 5. Конфигурация ядра sing-box
Возьмите файл шаблон конфигурации config.json из репозитория и заполните outbound-ы самостоятельно, также удалите/добавьте правила маршутизации, по умолчанию файлы с доменами для обхода берутся из этого репозитория, учитывайте доступность `raw.githubusercontent.com` с устройства которое настраиваете, затем замените файл в `/etc/sing-box/config.json`, далее включите службу, настройте автозапуск и запустите ее. 
В `/etc/config/sing-box` выставите параметры:

	option enabled '1'
    option user 'root'

#### 6. Запуск ядра 
	service sing-box enable && service sing-box start
### Дополнительно

 Просмотр логов ядра, при включенном логировании в /etc/sing-box/config.json:
 `logread -f -e sing-box`

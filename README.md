# HTTPs-CoDeSys
CoDeSys HTTPs


1. Форматируем флешку после Linux: diskpart, list disk, select disk X, clean, create partition primary, format quick, assign
2. Ставим обычную Raspbian 32bit (от 2020-02-13, но не от 2022!). Или еще можно - Raspberry Pi OS (Legasy) Lite (A port of Debian Bullseye wuth security update and no desktop enviroment). Логин/Пароль: pi, raspberry (можно и без раб.стола). В новой версии PiImager необходимо сразу задавать первичные параметры (SSH, username и проч.). И если там задать Логин/Пароль, то сразу можно подключиться удаленно, а иначе придется подключать клаву/мышь для задания Логина/Пароля.
3. Передергиваем флешку для перезапуска
4. Создаем файл ssh без расширения на флешке boot. После работы Raspbian этот файл исчезнет. (В новых версиях это можно не далать, а задать при накатывании образа).
5. У плат Rpi1, Rpi2 и др. возможен обмен serial-115200-8n1. Пины 4 и 5. Правка config.txt (необязательно): enable_uart=1. Потом через Putty.
6. У старых обрзов Raspы обновление длеается через sudo apt update, sudo apt upgrade
7. Для дисплея 800*480 надо поставить драйвер (необходимо подключение к Интернет):
	sudo rm -rf LCD-show
	git clone https://github.com/goodtft/LCD-show.git
	chmod -R 755 LCD-show
	cd LCD-show/
	sudo ./MPI4008-show
	- перезагрузка произойдет автоматически... Иногда нужно передернуть питание :)
	- следующий пункт тогда делать не надо.
8. Добавляем в ../../boot/config.txt (еще при создании SD-карты):
	# Дисплей
	hdmi_force_hotplug=1
	max_usb_current=1
	hdmi_drive=1
	hdmi_group=2
	hdmi_mode=1
	hdmi_mode=87
	hdmi_cvt 1024 600 60 6 0 0 0 (Разрешение нужно править под экран!)
	# Поворот экрана
	display_rotate=2 (иногда lcd_rotate=2)
	# Что-бы не показывать радужный экран при старте
	disable_splash=1
	# Разгон системы RPi3
	arm_freq=1300
	core_freq=500
	sdram_freq=500
	over_voltage=6
9. Поворачиваем тачскрин (необходимо подключение к Интернет):
	Установить libinput: sudo apt-get install xserver-xorg-input-libinput
	Код установки: sudo apt install xserver-xorg-input-synaptics
	Создать каталог: sudo mkdir /etc/X11/xorg.conf.d
	Скопировать файл: sudo cp /usr/share/X11/xorg.conf.d/40-libinput.conf /etc/X11/xorg.conf.d/
	Отредактировать файл: sudo nano /etc/X11/xorg.conf.d/40-libinput.conf
	Найти часть "libinput touchscreen catchall", добавить внутрь следующую инструкцию после MatchIsTouchscreen "on":
	90 градусов: Option "CalibrationMatrix" "0 1 0 -1 0 1 0 0 1"
	180 градусов: Option "CalibrationMatrix" "-1 0 1 0 -1 1 0 0 1"
	270 градусов: Option "CalibrationMatrix" "0 -1 1 1 0 0 0 0 1"
10. Статический IP-адрес (только в Linux):
	sudo nano /etc/dhcpcd.conf
	в конце: nodhcp interface eth0 static ip_address=192.168.0.243/24
	static routers=192.168.0.1 static domain_name_servers=192.168.0.1
11. Подключаем Вай-Фай (еще при создании SD-карты). Создаем файл: /boot/wpa_supplicant.conf. (Это не сработало для Raspberry Pi OS (Legasy) Lite) Пишем в него:
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1
	network={
	ssid="Guest"
	psk="Nero12345"
	}
12. Подключаем Вай-Фай (уже в Linux) (можно и через sudo raspi-config):
	sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
	Добавляем в конец:
	network={
	ssid="..."
	psk="..."
	}
	Делаем sudo reboot
13. Ставим удаленный рабочий стол (только если установлена система без стола!):
	sudo apt-get install tightvncserver xrdp
	(Необходимо подключение к Интернет, работает по проту 3389, запускается через Win+R, mstsc)
	(Или воспользуемся уже имеющимся без установки: (через ssh) sudo raspi-config, включить vnc, потом в терминале vncserver)
14. Устанавливаем mDNS (после чего веб-визуализация будет по адресу - myname:8080)  (В новых версиях это можно не далать, а задать
        при накатывании образа):
	sudo apt-get update (необходимо подключение к Интернет. Возможно со второго раза...)
	sudo apt-get install libnss-mdns - часто уже установлена
	sudo nano /etc/hostname - вписать новое имя вместо "raspberry"
	sudo nano /etc/hosts - вписать новое имя вместо "raspberry" в последней строке
	sudo /etc/init.d/hostname - обычно тут ошибка
	sudo reboot
15. Работа с камерой (usb не поддерживается!).
	Включить камеру в sudo raspi-config
	В старых системах: sudo apt update, а потом sudo apt-get update
	sudo apt-get update
	sudo apt-get dist-upgrade (очень длительная операция!)
	необязательно: sudo rpi-update
	sudo reboot
	git clone https://github.com/silvanmelchior/RPi_Cam_Web_Interface.git
	cd RPi_Cam_Web_Interface
	chmod u+x install.sh
	sudo bash install.sh  (очень длительная операция!)
	Потом смотришь фотки http://my_IP:my_PORT/html
        Потом смотришь тупо превью http://my_IP:my_port/html/min.php
16. Полное удаленное управление через interScada (из двух локальных сетей и динамическими серыми адресами) (использует порт 8088!, который можно сменить в настройках) (требуется установка программ intraScada на оба устройства):
	(требуется установка на рабочий комп с CodeSys ихнего IDE: intrascada.com/ru/product/desktop-app/)
	(требуется проброс портов 22(ssh), 1217(gateway - возможен конфликт), 8080(http), 11740(codesys), 5900(vnc))
	sudo apt update
	curl -sL https://git.io/JYAeq | sudo -E bash -s ru  (очень длительная операция!)
	- необходимо скопировать логин/пароль/веб-адрес (обычно admin, 202020, IP:8088/admin). Потом (еще в своей сети) узнать ключ p2p для удаленной сети в IDE (по ключу вход - https://p2p.ih-systems.com/).
17. Через приложение CodeSYS устанавливаем в систему Raspbian объект GateWay и агент RunTime (Можно через interScada).
     Если нет ответа от runTime, то sudo reboot и sudo service codesyscontrol restart. Необходимо будет создать своего пользователя (обычно pi, pi3)
18. Несколько прав меняем через ssh:
	Изменяем права доступа к файлу агента (0777?): sudo chmod 0777 /etc/CODESYSControl_User.cfg
	Добавляем в файл /etc/CODESYSControl_User.cfg:	[SysProcess] Command=AllowAll (типа все команды на исполнение. А можно Command.0=reboot Command.1=echo)
	Убираем управление пользователями в codesys в файле /etc/CODESYSControl_User.cfg: [CmpUserMgr] SECURITY.UserMgmtEnforce=NO
	Для разрешения скачивать файлы меняем в файле /etc/CODESYSControl_User.cfg: [CmpWebServerHandlerV3] AllowFileTransferServices=1,
		а путь к файлу '$$PlcLogic$$/Application/Manual_V1.pdf', тогда он будет падать в папку прямо из дерева проекта. Можно
		также указывать $$visu$$. А свои пути можно прописывать так в файле /etc/CODESYSControl_User.cfg: [SysFile] PlaceholderFilePath.1=/home/pi, $my_new_path$

	Права доступа на папку исполнения CodeSys: sudo chmod -R 0777 /var/opt/codesys
	Изменить порт визуализации: /etc/CODESYSControl.cfg: [CmpWebServer] WebServerPortNr=80 или WebServerSecurePortNr=443
	Если используешь стандартный uart-RPi, то нужно запретить вход в оболочку в raspi-config (38600 б/с). А хардовый порт нужно оставить!
	Если включаешь файл Питона в проект, то ему надо дать права на запуск: sudo chmod 0777 /var/opt/codesys/PlcLogic/Application/file.py
	Необходима перезагрузка: sudo reboot
	(рабочая папка: /var/opt/codesys)
	(загруженные в проект файлы ложаться сюда: /var/opt/codesys/PlcLogic/Application)
	(фавикон для визуализации (можно положить уже в CodeSys): /var/opt/codesys/PlcLogic/visu/favicon.ico)
	
19. Настройка FTP: sudo apt update; sudo apt install vsftpd; sudo systemctl status vsftpd;
	sudo nano /etc/vsftpd.conf (anonymous_enable=NO local_enable=YES write_enable=YES ssl_enable=YES);
	sudo systemctl restart vsftpd;
	
20. Настройка HTTPS: /etc/CODESYSControl.cfg :[CmpWebServer] ConnectionType=3 WebServerPortNr=80 WebServerSecurePortNr=443. Сгенерировать сертификат через SequreAgent.
	в PLC Shell: cert-gendhparams 1024 Тогда он будет работать хотя бы с IE и Firefox. Об этом: https://forge.codesys.com/forge/talk/Runtime/thread/0451764381/

21. Автоматический перезапуск агента.
	Добавляем в файл /etc/CODESYSControl_User.cfg:
	[SysProcess] 
	Command=AllowAll
	Добавляем в проект библиотеку SysProcess.
	Пишем в проекте:
	pResult: POINTER TO SysProcess.SysTypes.RTS_IEC_RESULT;
	SysProcess.SysProcessExecuteCommand('sudo service codesyscontrol restart', pResult);
	
22. Подключение комуникационного порта через свисток usb-rs485 (возможно придется перебирать ком.порты в IDE). При этом этого порта небудет в дереве проекта.
	sudo nano /etc/CODESYSControl_User.cfg
	[SysCom]
	Linux.Devicefile=/dev/ttyUSB (AMA?)   Linux.Devicefile=/dev/ttyS
	portnum := COM.SysCom.SYS_COMPORT1;
	Для множества свистков надо еще это: Сделать папку /etc/udev/rules.d доступной для записи: sudo chmod 0777 /etc/udev/rules.d
				Создать в этой папке файл: sudo nano /etc/udev/rules.d/99-usb-serial.rules
				Добавить в него: SUBSYSTEM=="tty", ATTRS{devpath}=="1.2", SYMLINK+="ttyUSB0" 
   				SUBSYSTEM=="tty", ATTRS{devpath}=="1.4", SYMLINK+="ttyUSB1" 
   				SUBSYSTEM=="tty", ATTRS{devpath}=="1.3", SYMLINK+="ttyUSB2" 
   				SUBSYSTEM=="tty", ATTRS{devpath}=="1.5", SYMLINK+="ttyUSB3"
	Перезагрузить: sudo reboot
	Проверить: lsusb
	
23. Проверено! Подключение комуникационного порта через свисток usb-cp2101 или usb-rs485. При этом этого порта небудет в дереве проекта. Свисток виден на ttyUSB0.
	sudo nano /etc/CODESYSControl_User.cfg
	[SysCom]
	Linux.Devicefile=/dev/ttyUSB
	portnum := COM.SysCom.SYS_COMPORT2; (без этой строки тоже работает)
        В проекте открываем порт 0 или 1.
	Перезагрузить: sudo reboot (Обязательно! Горячее подключение не допускается)
        При этом обычный uart не получилось запустить. Нужно больше времени.
	Проверить: lsusb. Там будет: Bus 001 Device 003: ID 10c4:ea60 Cygnal Integrated Products, Inc. CP2102/CP2109 UART Bridge Controller [CP210x family]
	
24. Автозапуск хромиума в режиме киоска: sudo nano /home/pi/.config/lxsession/LXDE-pi/autostart
	(или sudo nano /etc/xdg/lxsession/LXDE-pi/autostart зависит от установленной версии RASBIAN)
	(нужно поменять название страницы в последней строке)
	lxpanel --profile LXDE-pi
	@pcmanfm --desktop --profile LXDE-pi
	@xscreensaver -no-splash
	@point-rpi
	@xset s off
	@xset s noblank
	@xset -dpms
	@chromium-browser --noerrdialogs --kiosk --incognito --disable-translate http://localhost:8080/webvisu.htm m
	
25. Убираем курсор с экрана (в удаленном рабочем столе он виден): sudo nano /etc/lightdm/lightdm.conf
	В секции [Seat:*]
	активируем команду: xserver-command = X -nocursor
	
26. Настройка RTC-DS1302 для локальных сетей (прописывается rc.local):
	https://github.com/wawanUnic/RTC_DS1302

27. Прописываем в автозагрузку программы...
	sudo nano /etc/profile или rc.local
	
28. Продлеваем жизнь карты памяти. Убираем логирование на карту памяти:
	sudo nano /etc/fstab:
	a. ...
	b. ...
	c. PARTUUID=afb93022-02  /               ext4    defaults,noatime,nodiratime  0       1
	d. tmpfs    /tmp               tmpfs   nodev,nosuid,nodiratime,size=256M              0    0
	e. tmpfs    /var/log           tmpfs   defaults,noatime,nosuid,mode=0755,size=50M     0    0
	f. tmpfs    /var/spool/mqueue  tmpfs   defaults,noatime,nosuid,mode=0700,size=20M     0    0
	g. tmpfs    /var/tmp           tmpfs   defaults,noatime,nosuid,size=256M              0    0
	h. a swapfile is not a swap partition, no line here
	i. #   use  dphys-swapfile swap[on|off]  for that
	
29. Продлеваем жизнь карты памяти. Убираем файл подкачки:
	a. sudo dphys-swapfile swapoff
	b. sudo systemctl disable dphys-swapfile
	c. sudo rm /var/swap
	
30. Делаем систему только для чтения:
	sudo raspi-config; PerfomanceOptions; OverlayFileSystem - yes; BootReadOnly - yes
	
26. Браузер Хромиум в режиме киоска:
	Создать каталоги и файл: sudo touch ~/.config/autostart/chromium.desktop
	Правим файл: sudo nano ~/.config/autostart/chromium.desktop
	Добавляем в него: [Desktop Entry]
		Encoding=UTF-8
		Name=Connect
		Comment=Checks internet connectivity
		Exec=/usr/bin/chromium-browser -incognito --noerrdialogs --kiosk http://your_address:8080/stand.htm
	Перезагружаем: sudo reboot
	
27. Для обмена по SSH не по паролю, а по ключю: Сгенерировать открытый и закрытый ключи типа EdDSA(255bit). Вставить подсказку в ключ.
	Создать папку .ssh в домашнем каталоге не под рутом: mkdir -p /home/user_name/.ssh
	Создать файл ключа: sudo nano /home/user_name/.ssh/authorized_keys
	Тупо вставить туда открытый скопипастенный ключ: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBhJnAsnzCXmpm02TauSnWTxWwcdgAO8AcLGXwgQ2s/6 eddsa-key-20220809
	Права: chmod 700 /home/user_name/.ssh
	Права: chmod 600 /home/user_name/.ssh/authorized_keys
	Права, если создавалось рутом: chown -R username:username /home/username/.ssh
	Закрытый ключ можно закрыть легким паролем и оставить на локальной машине.
	Отключить возможность входа по поролю "sudo nano /etc/ssh/sshd_config" опцией "PasswordAuthentication no"
	Перезагружаемся. Отключение действует на все учетные записи.

28. Вынесем волатильные разделы на виртуальные диски
	Например, /tmp и /var/log, для того, чтобы:
	система не точила попусту sd-карту,
	было меньше шансов, что при аварийном отключении питания развалится корневая файловая система,
	чтобы в отдаленном будущем логи не забили свободное место на карте.
	Для этого в /etc/fstab добавим строки:
	tmpfs           /var/log        tmpfs   defaults,noatime,nosuid,mode=0755,size=100m    0 0
	tmpfs           /tmp            tmpfs   defaults,noatime,nosuid,size=100m    0 0
	Не забудьте поставить в конце файла перевод строки.
	Здесь разделам /var/log и /tmp выделено по 100 мб памяти (она будет использоваться только по мере заполнения). Гигабайтному Pi, который используется только для RDP, эти траты совершенно ничем не помешают.
	Перезагрузим систему, а затем выполним команду:
	# df
	и убедимся, что разделы смонтировались на tmpfs (3 и 4 колонки могут отличаться):
	tmpfs             102400       8    102392   1% /tmp
	tmpfs             102400     224    102176   1% /var/log

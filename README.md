# Скрипты для Linux

## Рандомизация LAN IP и LAN-MAC-адресов на роутере с OpenWRT

1. Подключиться к роутеру через SSH

2. Создать файл identifiers_randomization:

```bash
#!/bin/sh /etc/rc.common

START=17

start() {
  #MAC-ADDRESSES RANDOMIZATION
  NEWMAC1=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for br-lan
  NEWMAC2=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for lan
  NEWMAC3=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for Wi-Fi0
  NEWMAC4=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for Wi-Fi1
  NEWMAC5=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for eth0
  NEWMAC6=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for lan1
  NEWMAC7=$(printf "%02x" $(( $(hexdump -n1 -e'/1 "0x%02x"' /dev/urandom) & ~1 | 2)) && hexdump -n5 -e'/1 ":%02x"' /dev/urandom) #Get new MAC-address for lan2
  uci set network.@device[0].macaddr=${NEWMAC1} #Set new MAC-address for br-lan
  uci set network.lan.macaddr=${NEWMAC2} #Set new MAC-address for lan
  uci set wireless.@wifi-iface[0].macaddr=${NEWMAC3} #Set new MAC-address for Wi-Fi0
  uci set wireless.@wifi-iface[1].macaddr=${NEWMAC4} #Set new MAC-address for Wi-Fi1
  uci set network.@device[1].macaddr=${NEWMAC5} #Set new MAC-address for eth0
  uci set network.@device[2].macaddr=${NEWMAC6} #Set new MAC-address for lan1
  uci set network.@device[3].macaddr=${NEWMAC7} #Set new MAC-address for lan2
  #LAN IP-ADDRESS RANDOMIZATION
  #random_number_for_ip=$(awk 'BEGIN{srand(); print int(rand()*245)+5}') #Get random number
  #random_ip1="192.168.$random_number_for_ip.1" #Get random IP
  #random_ip2="$random_ip1/24" #Add subnet mask to IP
  #uci set network.lan.ipaddr=${random_ip2} #Set new IP-address
  #HOSTNAME RANDOMIZATION
  line_number=$(awk 'BEGIN{srand(); print int(rand()*3150)+1}') #Get random line number
  new_hostname=$(sed -n "${line_number}p" /usr/local/bin/hostnames.txt) #Get random hostname
  uci set system.@system[0].hostname=${new_hostname} #Set new hostname
  uci set system.cfg01e48a.hostname=${new_hostname} #Set new hostname
  #SET NEW SETTINGS
  uci commit network
  uci commit wireless
  uci commit system
  /etc/init.d/network restart
  /etc/init.d/system restart
}
```
3. Создать файл, содержащий по одному имени устройства в каждой строке

4. Скопировать файлы в необходимые директории и включить скрипт:

```bash

#!/bin/bash

cp identifiers_randomization /etc/init.d/identifiers_randomization

chmod +x /etc/init.d/identifiers_randomization

cp hostnames.txt /usr/local/bin/hostnames.txt

/etc/init.d/identifiers_randomization enable

```


## Рандомизация MAC-адреса

1. Создать файл 99-macchanger.conf:

```bash

[device]
wifi.scan-rand-mac-address=yes

[connection]
wifi.cloned-mac-address=random
ethernet.cloned-mac-address=random
connection.stable-id=/

```
2. Скопировать файл в директорию /etc/NetworkManager/conf.d/ и перезагрузить NetworkManager

```bash

#!/bin/bash

cp 99-macchanger.conf /etc/NetworkManager/conf.d/99-macchanger.conf

systemctl restart NetworkManager

```

## Рандомизация имени ПК при перезагрузке

1. Создать файл hotnames.txt, содержащий по одному имени в каждой строке:

2. Создать файл randomize_hostname.sh:

```bash

#!/bin/bash

HOSTNAMES_FILE="/usr/local/bin/randomize_hostname/hostnames.txt"

num_lines=$(wc -l < "$HOSTNAMES_FILE")

random_line=$((RANDOM % num_lines + 1))

new_hostname=$(sed -n "${random_line}p" "$HOSTNAMES_FILE")

echo "Setting new hostname: $new_hostname"

hostnamectl set-hostname $new_hostname

```

3. Создать файл randomize_hostname.service:

```bash

[Unit]
Description=Randomize Hostname
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/randomize_hostname/randomize_hostname.sh

[Install]
WantedBy=multi-user.target
```

4. Скопировать файлы в необходимые директории и включить скрипт:

```bash
#!/bin/bash

sudo mkdir /usr/local/bin/randomize_hostname

sudo cp randomize_hostname.sh /usr/local/bin/randomize_hostname/randomize_hostname.sh
sudo chmod +x /usr/local/bin/randomize_hostname/randomize_hostname.sh

sudo cp hostnames.txt /usr/local/bin/randomize_hostname/hostnames.txt

sudo cp randomize_hostname.service /etc/systemd/system/randomize_hostname.service

sudo systemctl daemon-reload
sudo systemctl enable randomize_hostname.service
```

## Рандомизация часового пояса при перезагрузке

1. Создать файл timezones.txt, содержащий по одному часовому поясу в каждой строке:

```bash

Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Asmera
Africa/Bamako
Africa/Bangui
Africa/Banjul
Africa/Bissau
Africa/Blantyre
Africa/Brazzaville
Africa/Bujumbura
Africa/Cairo
Africa/Casablanca
Africa/Ceuta
Africa/Conakry
Africa/Dakar
Africa/Dar_es_Salaam
Africa/Djibouti
Africa/Douala
Africa/El_Aaiun
Africa/Freetown
Africa/Gaborone
Africa/Harare
Africa/Johannesburg
Africa/Juba
Africa/Kampala
Africa/Khartoum
Africa/Kigali
Africa/Kinshasa
Africa/Lagos
Africa/Libreville
Africa/Lome
Africa/Luanda
Africa/Lubumbashi
Africa/Lusaka
Africa/Malabo
Africa/Maputo
Africa/Maseru
Africa/Mbabane
Africa/Mogadishu
Africa/Monrovia
Africa/Nairobi
Africa/Ndjamena
Africa/Niamey
Africa/Nouakchott
Africa/Ouagadougou
Africa/Porto-Novo
Africa/Sao_Tome
Africa/Timbuktu
Africa/Tripoli
Africa/Tunis
Africa/Windhoek
America/Adak
America/Anchorage
America/Anguilla
America/Antigua
America/Araguaina
America/Argentina/Buenos_Aires
America/Argentina/Catamarca
America/Argentina/ComodRivadavia
America/Argentina/Cordoba
America/Argentina/Jujuy
America/Argentina/La_Rioja
America/Argentina/Mendoza
America/Argentina/Rio_Gallegos
America/Argentina/Salta
America/Argentina/San_Juan
America/Argentina/San_Luis
America/Argentina/Tucuman
America/Argentina/Ushuaia
America/Aruba
America/Asuncion
America/Atikokan
America/Atka
America/Bahia
America/Bahia_Banderas
America/Barbados
America/Belem
America/Belize
America/Blanc-Sablon
America/Boa_Vista
America/Bogota
America/Boise
America/Buenos_Aires
America/Cambridge_Bay
America/Campo_Grande
America/Cancun
America/Caracas
America/Catamarca
America/Cayenne
America/Cayman
America/Chicago
America/Chihuahua
America/Ciudad_Juarez
America/Coral_Harbour
America/Cordoba
America/Costa_Rica
America/Creston
America/Cuiaba
America/Curacao
America/Danmarkshavn
America/Dawson
America/Dawson_Creek
America/Denver
America/Detroit
America/Dominica
America/Edmonton
America/Eirunepe
America/El_Salvador
America/Ensenada
America/Fort_Nelson
America/Fort_Wayne
America/Fortaleza
America/Glace_Bay
America/Godthab
America/Goose_Bay
America/Grand_Turk
America/Grenada
America/Guadeloupe
America/Guatemala
America/Guayaquil
America/Guyana
America/Halifax
America/Havana
America/Hermosillo
America/Indiana/Indianapolis
America/Indiana/Knox
America/Indiana/Marengo
America/Indiana/Petersburg
America/Indiana/Tell_City
America/Indiana/Vevay
America/Indiana/Vincennes
America/Indiana/Winamac
America/Indianapolis
America/Inuvik
America/Iqaluit
America/Jamaica
America/Jujuy
America/Juneau
America/Kentucky/Louisville
America/Kentucky/Monticello
America/Knox_IN
America/Kralendijk
America/La_Paz
America/Lima
America/Los_Angeles
America/Louisville
America/Lower_Princes
America/Maceio
America/Managua
America/Manaus
America/Marigot
America/Martinique
America/Matamoros
America/Mazatlan
America/Mendoza
America/Menominee
America/Merida
America/Metlakatla
America/Mexico_City
America/Miquelon
America/Moncton
America/Monterrey
America/Montevideo
America/Montreal
America/Montserrat
America/Nassau
America/New_York
America/Nipigon
America/Nome
America/Noronha
America/North_Dakota/Beulah
America/North_Dakota/Center
America/North_Dakota/New_Salem
America/Nuuk
America/Ojinaga
America/Panama
America/Pangnirtung
America/Paramaribo
America/Phoenix
America/Port-au-Prince
America/Port_of_Spain
America/Porto_Acre
America/Porto_Velho
America/Puerto_Rico
America/Punta_Arenas
America/Rainy_River
America/Rankin_Inlet
America/Recife
America/Regina
America/Resolute
America/Rio_Branco
America/Rosario
America/Santa_Isabel
America/Santarem
America/Santiago
America/Santo_Domingo
America/Sao_Paulo
America/Scoresbysund
America/Shiprock
America/Sitka
America/St_Barthelemy
America/St_Johns
America/St_Kitts
America/St_Lucia
America/St_Thomas
America/St_Vincent
America/Swift_Current
America/Tegucigalpa
America/Thule
America/Thunder_Bay
America/Tijuana
America/Toronto
America/Tortola
America/Vancouver
America/Virgin
America/Whitehorse
America/Winnipeg
America/Yakutat
America/Yellowknife
Antarctica/Casey
Antarctica/Davis
Antarctica/DumontDUrville
Antarctica/Macquarie
Antarctica/Mawson
Antarctica/McMurdo
Antarctica/Palmer
Antarctica/Rothera
Antarctica/South_Pole
Antarctica/Syowa
Antarctica/Troll
Antarctica/Vostok
Arctic/Longyearbyen
Asia/Aden
Asia/Almaty
Asia/Amman
Asia/Anadyr
Asia/Aqtau
Asia/Aqtobe
Asia/Ashgabat
Asia/Ashkhabad
Asia/Atyrau
Asia/Baghdad
Asia/Bahrain
Asia/Baku
Asia/Bangkok
Asia/Beirut
Asia/Bishkek
Asia/Brunei
Asia/Calcutta
Asia/Chita
Asia/Choibalsan
Asia/Chongqing
Asia/Chungking
Asia/Colombo
Asia/Dacca
Asia/Damascus
Asia/Dhaka
Asia/Dili
Asia/Dubai
Asia/Dushanbe
Asia/Famagusta
Asia/Gaza
Asia/Harbin
Asia/Hebron
Asia/Ho_Chi_Minh
Asia/Hong_Kong
Asia/Hovd
Asia/Istanbul
Asia/Jakarta
Asia/Jayapura
Asia/Jerusalem
Asia/Kabul
Asia/Karachi
Asia/Kashgar
Asia/Kathmandu
Asia/Katmandu
Asia/Khandyga
Asia/Kolkata
Asia/Kuala_Lumpur
Asia/Kuching
Asia/Kuwait
Asia/Macao
Asia/Macau
Asia/Makassar
Asia/Manila
Asia/Muscat
Asia/Nicosia
Asia/Phnom_Penh
Asia/Pontianak
Asia/Qatar
Asia/Qyzylorda
Asia/Rangoon
Asia/Riyadh
Asia/Saigon
Asia/Seoul
Asia/Shanghai
Asia/Singapore
Asia/Taipei
Asia/Tashkent
Asia/Tbilisi
Asia/Tehran
Asia/Tel_Aviv
Asia/Thimbu
Asia/Thimphu
Asia/Tokyo
Asia/Ujung_Pandang
Asia/Ulaanbaatar
Asia/Ulan_Bator
Asia/Urumqi
Asia/Vientiane
Asia/Yangon
Asia/Yerevan
Atlantic/Azores
Atlantic/Bermuda
Atlantic/Canary
Atlantic/Cape_Verde
Atlantic/Faeroe
Atlantic/Faroe
Atlantic/Jan_Mayen
Atlantic/Madeira
Atlantic/Reykjavik
Atlantic/South_Georgia
Atlantic/St_Helena
Atlantic/Stanley
Australia/ACT
Australia/Adelaide
Australia/Brisbane
Australia/Broken_Hill
Australia/Canberra
Australia/Currie
Australia/Darwin
Australia/Eucla
Australia/Hobart
Australia/LHI
Australia/Lindeman
Australia/Lord_Howe
Australia/Melbourne
Australia/NSW
Australia/North
Australia/Perth
Australia/Queensland
Australia/South
Australia/Sydney
Australia/Tasmania
Australia/Victoria
Australia/West
Australia/Yancowinna
Brazil/Acre
Brazil/DeNoronha
Brazil/East
Brazil/West
CET
CST6CDT
Canada/Atlantic
Canada/Central
Canada/Eastern
Canada/Mountain
Canada/Newfoundland
Canada/Pacific
Canada/Saskatchewan
Canada/Yukon
Chile/Continental
Chile/EasterIsland
Cuba
EET
EST
EST5EDT
Egypt
Eire
Etc/GMT
Etc/GMT+0
Etc/GMT+1
Etc/GMT+10
Etc/GMT+11
Etc/GMT+12
Etc/GMT+2
Etc/GMT+4
Etc/GMT+5
Etc/GMT+6
Etc/GMT+7
Etc/GMT+8
Etc/GMT+9
Etc/GMT-0
Etc/GMT-1
Etc/GMT-10
Etc/GMT-11
Etc/GMT-12
Etc/GMT-13
Etc/GMT-14
Etc/GMT-2
Etc/GMT-3
Etc/GMT-4
Etc/GMT-5
Etc/GMT-6
Etc/GMT-7
Etc/GMT-8
Etc/GMT-9
Etc/GMT0
Etc/Greenwich
Etc/UCT
Etc/UTC
Etc/Universal
Etc/Zulu
Europe/Amsterdam
Europe/Andorra
Europe/Athens
Europe/Belfast
Europe/Belgrade
Europe/Berlin
Europe/Bratislava
Europe/Brussels
Europe/Bucharest
Europe/Budapest
Europe/Busingen
Europe/Chisinau
Europe/Copenhagen
Europe/Dublin
Europe/Gibraltar
Europe/Guernsey
Europe/Helsinki
Europe/Isle_of_Man
Europe/Istanbul
Europe/Jersey
Europe/Lisbon
Europe/Ljubljana
Europe/London
Europe/Luxembourg
Europe/Madrid
Europe/Malta
Europe/Mariehamn
Europe/Monaco
Europe/Nicosia
Europe/Oslo
Europe/Paris
Europe/Podgorica
Europe/Prague
Europe/Riga
Europe/Rome
Europe/San_Marino
Europe/Sarajevo
Europe/Skopje
Europe/Sofia
Europe/Stockholm
Europe/Tallinn
Europe/Tirane
Europe/Tiraspol
Europe/Vaduz
Europe/Vatican
Europe/Vienna
Europe/Vilnius
Europe/Warsaw
Europe/Zagreb
Europe/Zurich
Factory
GB
GB-Eire
GMT
GMT+0
GMT-0
GMT0
Greenwich
HST
Hongkong
Iceland
Indian/Antananarivo
Indian/Chagos
Indian/Christmas
Indian/Cocos
Indian/Comoro
Indian/Mahe
Indian/Maldives
Indian/Mauritius
Indian/Mayotte
Indian/Reunion
Iran
Israel
Jamaica
Kwajalein
Libya
MET
MST
MST7MDT
Mexico/BajaNorte
Mexico/BajaSur
Mexico/General
NZ-CHAT
Navajo
PRC
PST8PDT
Pacific/Apia
Pacific/Auckland
Pacific/Bougainville
Pacific/Chatham
Pacific/Chuuk
Pacific/Easter
Pacific/Efate
Pacific/Enderbury
Pacific/Fakaofo
Pacific/Fiji
Pacific/Funafuti
Pacific/Galapagos
Pacific/Gambier
Pacific/Guadalcanal
Pacific/Guam
Pacific/Honolulu
Pacific/Johnston
Pacific/Kanton
Pacific/Kiritimati
Pacific/Kosrae
Pacific/Kwajalein
Pacific/Majuro
Pacific/Marquesas
Pacific/Midway
Pacific/Nauru
Pacific/Niue
Pacific/Norfolk
Pacific/Noumea
Pacific/Pago_Pago
Pacific/Palau
Pacific/Pitcairn
Pacific/Pohnpei
Pacific/Ponape
Pacific/Port_Moresby
Pacific/Rarotonga
Pacific/Saipan
Pacific/Samoa
Pacific/Tahiti
Pacific/Tarawa
Pacific/Tongatapu
Pacific/Truk
Pacific/Wake
Pacific/Wallis
Pacific/Yap
Poland
Portugal
ROC
ROK
Singapore
Turkey
UCT
US/Alaska
US/Aleutian
US/Arizona
US/Central
US/East-Indiana
US/Eastern
US/Hawaii
US/Indiana-Starke
US/Michigan
US/Mountain
US/Pacific
US/Samoa
UTC
Universal
W-SU
WET
```

2. Создать файл randomize_timezone.sh:

```bash

#!/bin/bash

TIMEZONES_FILE="/usr/local/bin/randomize_timezone/timezones.txt"

num_lines1=$(wc -l < "$TIMEZONES_FILE")

random_line1=$((RANDOM % num_lines1 + 1))

new_timezone=$(sed -n "${random_line1}p" "$TIMEZONES_FILE")

timedatectl set-timezone $new_timezone

```

3. Создать файл randomize_timezone.service:

```bash

[Unit]
Description=Randomize Timezone
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/randomize_timezone/randomize_timezone.sh

[Install]
WantedBy=multi-user.target
```

4. Скопировать файлы в необходимые директории и включить скрипт:

```bash
#!/bin/bash

mkdir /usr/local/bin/randomize_timezone

cp randomize_timezone.sh /usr/local/bin/randomize_timezone/randomize_timezone.sh
chmod +x /usr/local/bin/randomize_timezone/randomize_timezone.sh

cp timezones.txt /usr/local/bin/randomize_timezone/timezones.txt

cp randomize_timezone.service /etc/systemd/system/randomize_timezone.service

systemctl daemon-reload
systemctl enable randomize_timezone.service
```

## Удаление скриншотов, сделанных больше одного дня назад, при перезагрузке

1. Создать файл clear_screenshots.sh:

```bash

#!/bin/bash

screenshots_dir="/home/name/Screenshots" # указать свою директорию для скриншотов
screencasts_dir="/home/name/Screencasts" # указать свою директорию для скринкастов

if [ -d "$screenshots_dir" ]; then
    find "$screenshots_dir" -type f -mtime +0 -exec wipe -f {} \;
else
    echo "Screenshots folder not found"
fi

if [ -d "$screencasts_dir" ]; then
    find "$screencasts_dir" -type f -mtime +0 -exec wipe -f {} \;
else
    echo "Screencasts folder not found"
fi

```

2. Создать файл clear_screenshots.service:

```bash

[Unit]
Description=Clear Screenshots
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/clear_screenshots/clear_screenshots.sh

[Install]
WantedBy=multi-user.target

```

3. Скопировать файлы в необходимые директории и включить скрипт:

```bash
#!/bin/bash

mkdir /usr/local/bin/clear_screenshots

cp clear_screenshots.sh /usr/local/bin/clear_screenshots/clear_screenshots.sh
chmod +x /usr/local/bin/clear_screenshots/clear_screenshots.sh

cp clear_screenshots.service /etc/systemd/system/clear_screenshots.service

systemctl daemon-reload
systemctl enable clear_screenshots.service

```

## Выключение Wi-Fi при закрытии крышки ноутбука

1. Создать файл disable_wifi_on_lid_close.sh:

```bash

#!/bin/bash

WIFI_STATE_FILE="/sys/class/rfkill/rfkill0/state"

disable_wifi() {
    echo "0" > "$WIFI_STATE_FILE"
    echo "Wi-Fi turned off."
}

while true; do
    if [ "$(cat /proc/acpi/button/lid/LID0/state | awk '{print $2}')" == "closed" ]; then
        disable_wifi
    fi
    sleep 1
done

```

2. Создать файл disable_wifi_on_lid_close.service:

```bash

[Unit]
Description=Wi-Fi Disable on Lid Close
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/switch_wifi/disable_wifi_on_lid_close.sh

[Install]
WantedBy=multi-user.target

```
3. Скопировать файлы в необходимые директории и включить скрипт:

```bash

#!/bin/bash

mkdir /usr/local/bin/switch_wifi

cp disable_wifi_on_lid_close.sh /usr/local/bin/switch_wifi/disable_wifi_on_lid_close.sh
chmod +x /usr/local/bin/switch_wifi/disable_wifi_on_lid_close.sh

cp disable_wifi_on_lid_close.service /etc/systemd/system/disable_wifi_on_lid_close.service

systemctl daemon-reload
systemctl enable /etc/systemd/system/disable_wifi_on_lid_close.service
```

## Включение Wi-Fi при открытии крышки ноутбука

1. Создать файл enable_wifi_on_lid_open.sh:

```bash

#!/bin/bash

WIFI_STATE_FILE="/sys/class/rfkill/rfkill0/state"

enable_wifi() {
    echo "1" > "$WIFI_STATE_FILE"
    echo "Wi-Fi turned on."
}

while true; do
    if [ "$(cat /proc/acpi/button/lid/LID0/state | awk '{print $2}')" == "open" ]; then
        enable_wifi
    fi
    sleep 1
done

```

2. Создать файл enable_wifi_on_lid_open.service:

```bash

[Unit]
Description=Wi-Fi Enable on Lid Open
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/switch_wifi/enable_wifi_on_lid_open.sh

[Install]
WantedBy=multi-user.target

```

## Добавление модулей ядра в черный список

```bash
#!/bin/bash

touch /etc/modprobe.d/blacklist.conf

echo "module_name" >> /etc/modprobe.d/blacklist.conf # module_name - имя модуля

```
## Добавление модулей ядра в черный список


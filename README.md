# smb_server
## Данная конфигурация протестирована на:
ОС: Ubuntu 22.04.4 LTS (5.15.0-125-generic)
samba server: 4.15.13-Ubuntu
## Настройка в ручную
Простой домашний smb server, минимум заморочек, максимум скорости настройки и развертывания.
### Используемые команды и приложения:
1. testparm -s -- проверяем конфигурацию на валидность
2. \* vim -- приложение для редактирования текстовых файлов
3. apt -- менеджер пакетов в linux.
4. systemctl restart smbd -- перезагрузка службы smb
5. systemctl status smbd -- проверяем состояние и наличие ошибок
6. smbpasswd -a guest -- добавляем существующего пользователя в ОС в базу smb
7. smbpasswd -e user -- включаем добавленного пользователя
8. mkdir -p -- создание каталога/ов, который будем публиковать в сети
9. chown -R -- настраиваем владельца и группу каталога
10. chmod -- настраиваем уровень доступа пользователя к каталогу.
### Пример конфигурационного файла smb.conf:
Открываем в текстовом редакторе файл конфигурации сервера:
`sudo vim /etc/samba/smb.conf`
```
; Глобальные настройки сервера
[global]
; Имя компьютера, которое будет отображаться в сетевом окружении
netbios name = main-server
server string =
; Рабочая группа клиентов
workgroup = WORKGROUP
socket options = TCP_NODELAY IPTOS_LOWDELAY SO_KEEPALIVE SO_RCVBUF=8192 SO_SNDBUF=8192
passdb backend = tdbsam
security = user
; Файл для альясов имен юзеров
username map = /etc/samba/smbusers
name resolve order = host wins bcast
; wins support устанавливается в yes, если ваш nmbd(8) в Самба является WINS сервером. Не устанавливайте этот параметр в yes если у вас нет нескольких подсетей и вы не хотите чтобы ваш nmbd работал как WINS сервер. Никогда не устанавливайте этот параметр в yes более чем на одной машине в пределах одной подсети.
wins support = no
; Поддержка принтеров
;printing = CUPS
;printcap name = CUPS
; Логи
log file = /var/log/samba/log.%m
; Настройка привязки к интерфейсам, на каких слушать, если не указано слушает на все интерфейсах
; interfaces = lo, eth0
; bind interfaces only = true
;
;[print$]
; path = /var/lib/samba/printers
; browseable = yes
; guest ok = yes
; read only = yes
; write list = root
; create mask = 0664
; directory mask = 0775
;
;[printers]
; path = /tmp
; printable = yes
; guest ok = yes
; browseable = no
;
;[DVD-ROM Drive]
;path = /media/cdrom
;browseable = yes
;read only = yes
;guest ok = yes
; Шара жесткого диска
; Имя шары, видно у клиентов
[<Название_публикуемого_каталога_которое_видит_подключившийся_пользователь>]
; Путь к расшариваемому диску
path = /home/<username>/<публикуемый_каталог>
; Можно ли просматривать
browseable = yes
read only = no
guest ok = no
create mask = 0644
directory mask = 0755
; Привязка к определенному имени пользователя или группе, имена через пробел
; force user = user1 user2
; force group = group1 group2
; Еще один жесткий диск, по аналогии с тем что выше
[Docs]
path = /home/neon/docs
browseable = yes
read only = no
guest ok = no
create mask = 0644
directory mask = 0755
```
### Настраиваем пользователя:
Предположим имя нашего в ОС user, который будет владельцев публикуемых каталогов. Добавим его в базу smb сервера и включим:
```
smbpasswd -a user
smbpasswd -e user
```
Далее необходимо создать алиасы, чтобы облегчить подключение к серверу с ОС Windows:
```
sudo vim /etc/samba/smbusers
```
Добавим в него следующие строки:
```
# Unix_name = SMB_name1 SMB_name2
user = Admin
```
### Запуск smb сервера:
```
sudo systemctl restart smbd
sudo systemctk status smbd
```
Настройка завершена.
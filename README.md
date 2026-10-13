# mephi-session-project-2026

Сессионный проект по администрированию Linux (RHEL-подобная ОС), МИФИ, 2026. В репозитории содержатся артефакты, подтверждающие выполнение настроек сетевой подсистемы, дискового пространства, пользователей, прав доступа, SELinux, PAM, SSH и веб-сервера Nginx.

## Что было настроено

Согласно истории команд (`project_history.txt`), на узле `mephi-2026.domain.local` были выполнены следующие шаги:

1. **Сетевые параметры и имя хоста**:
   - Назначено имя хоста `mephi-2026.domain.local` через `hostnamectl`.
   - Для сетевого соединения `Wired connection 1` через `nmcli` настроен статический IPv4-адрес `192.168.1.100/24`, шлюз `192.168.1.1` и DNS `8.8.8.8`.
   - Проверена доступность шлюза и внешнего адреса утилитой `ping`.

2. **Управление пакетами и утилитами**:
   - Установлены пакеты `nginx`, `tcpdump`, `libcap-ng-utils` через `dnf`.
   - Загружен и повторно развернут RPM-пакет утилиты `tcpdump`.

3. **Дисковая подсистема и файловые системы**:
   - Создан раздел `/dev/sdb1` на диске `/dev/sdb` через `fdisk`.
   - Создана файловая система `ext4` с меткой тома `MEPHI_DATA` (`mkfs.ext4 -L MEPHI_DATA /dev/sdb1`).
   - Создана точка монтирования `/data/mephi-web`.
   - Добавлена запись автоподключения в `/etc/fstab` по метке `LABEL=MEPHI_DATA` и выполнено монтирование.

4. **Пользователи и группы**:
   - Создан пользователь `mephi-admin` и группа `mephi-devs`.
   - Пользователь `mephi-admin` включен в дополнительную группу `mephi-devs`.
   - На директорию `/data/mephi-web` назначены владелец `mephi-admin`, группа `mephi-devs`, права доступа `2775` (включая SGID).

5. **Безопасность (SELinux, Capabilities, PAM, SSH)**:
   - Проверен статус SELinux (`Enforcing`).
   - Для каталога `/data/mephi-web(/.*)?` добавлен контекст `httpd_sys_content_t` через `semanage` и применен с помощью `restorecon`.
   - С бинарного файла `/usr/sbin/tcpdump` снят SUID-бит, установлены файловые привилегии (capabilities) `cap_net_raw,cap_net_admin+ep`.
   - Создан список `/etc/denied_users` со значением `root`.
   - В PAM-конфигурации `/etc/pam.d/password-auth` и `/etc/pam.d/sshd` подключен модуль `pam_listfile.so` для блокировки учетной записи `root`.
   - В `/etc/ssh/sshd_config` прописана директива `PermitRootLogin no`, служба `sshd` перезапущена.

6. **Веб-сервер Nginx**:
   - Корневая директория сайта в `/etc/nginx/nginx.conf` переведена с `/usr/share/nginx/html` на `/data/mephi-web`.
   - Создан файл `/data/mephi-web/index.html` с содержимым `Hello from Student: 368619`.
   - Служба `nginx` активирована в автозагрузке и перезапущена; корректность отдачи проверена локальным запросом через `curl http://localhost`.

## Описание файлов репозитория

- `curl_output.txt` - результат локального HTTP-запроса (`curl http://localhost`), возвращающий `Hello from Student: 368619`.
- `file_contexts.txt` - подтверждение назначения SELinux-контекста `system_u:object_r:httpd_sys_content_t:s0` для пути `/data/mephi-web`.
- `fstab.txt` - конфигурационный файл `/etc/fstab` с записью монтирования раздела `LABEL=MEPHI_DATA` в точку `/data/mephi-web` с типом `ext4`.
- `index.html` - исходный файл веб-страницы со строкой идентификатора студента `368619`.
- `mephi-nginx-screenshot.png` - скриншот страницы, которую отдаёт nginx.
- `network_check.txt` - вывод команд `ping` до шлюза `192.168.1.1` и внешнего DNS `8.8.8.8`, подтверждающий отсутствие потерь пакетов.
- `nginx_recent_logs.txt` - журнал `journalctl` службы `nginx.service`, подтверждающий успешное тестирование конфигурации и перезапуск сервиса на хосте `mephi-2026.domain.local`.
- `permissions.txt` - вывод команды `stat` для директории `/data/mephi-web`, подтверждающий права доступа `2775/drwxrwsr-x`, владельца `mephi-admin` (UID 1001), группу `mephi-devs` (GID 1002) и SELinux-контекст.
- `project_history.txt` - полный лог команд bash, выполненных в процессе настройки системы.
- `selinux_status.txt` - вывод `getenforce`, подтверждающий рабочий режим SELinux `Enforcing`.
- `tcpdump_capabilities.txt` - вывод команды `getcap /usr/sbin/tcpdump`, подтверждающий наличие привилегий `cap_net_admin,cap_net_raw=ep`.
- `tcpdump.rpm` - сохраненный установочный пакет утилиты `tcpdump`.
- `users_groups.txt` - вывод команд проверки членства пользователя `mephi-admin` (UID 1001) в группах `mephi-admin` (1001) и `mephi-devs` (1002).

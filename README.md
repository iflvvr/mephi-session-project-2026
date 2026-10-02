# Безопасность GNU/Linux: настройка базовых средств защиты
ОС РЕД ОС 8 без графического интерфейса

### Раздел 1. Установка дистрибутива

|||
|---|---|
| 1.1 Сеть (DHCP) | В установщике: метод IPv4 «Автоматически (DHCP)», включено автоподключение |
| 1.2 Имя хоста | В установщике задано `mephi-2026.domain.local` |
| 1.3 Проверка связи | `ping -c 4 8.8.8.8 > ~/ping.out` |

### Раздел 2. Управление программным обеспечением

|||
|---|---|
| 2.1 Обновление | `dnf update -y`, затем перезагрузка на новое ядро |
| 2.2 Пакеты из репозиториев | `dnf install -y nginx libcap-ng-utils` |
| 2.3 Локальный RPM | `dnf download tcpdump --destdir=/tmp`, затем `rpm -ivh /tmp/tcpdump*.rpm`; история: `dnf history > ~/dnf.out` |

> Примечание: `dnf download` и `rpm -ivh` не создают транзакций в `dnf history`, поэтому tcpdump в `dnf.out` нет. Его установка видна в `history.out`.

### Раздел 3. Файловые системы

|||
|---|---|
| 3.1 Файловая система | Второй диск `/dev/vdb`. `parted -s /dev/vdb mklabel gpt mkpart primary ext4 0% 100%`, затем `mkfs.ext4 -L MEPHI_WEB /dev/vdb1` |
| 3.2 Монтирование | `mkdir /mephi-web`; в `/etc/fstab` добавлена строка `LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2`; ручное монтирование `mount /mephi-web` |

### Раздел 4. Управление сервисами

|||
|---|---|
| 4.1 Веб-сервер | Запуск и автозапуск при загрузке `systemctl enable --now nginx` |
| 4.2 Журналирование | `journalctl -u nginx -b > ~/journalctl.out` |

### Раздел 5. Управление доступом

|||
|---|---|
| 5.1 DAC | Группы `team` и `curators` (GID 4444). Пользователи `user1`, `user2`, `user3` (UID 5501, 5502, 5503, основная группа `team`). Кураторы `curator1`, `curator2` (основная группа `curators`). Каталог `/data/mephi-2026`: владелец `root:team`, права `2770` (setgid наследует группу `team`, остальным доступ закрыт). ACL: для `curators` заданы ACL и default ACL (только чтение, `r-x`). Права `team` обеспечены владением группы и `setgid`: новые файлы наследуют группу `team` и доступны ей на чтение и запись |
| 5.2 Привилегии | Вместо set-UID бинарнику выданы capabilities: `setcap cap_net_admin,cap_net_raw=eip /usr/sbin/tcpdump`. Проверка: `sudo -u user1 tcpdump -c 3 -i any` |
| 5.3 MAC | SELinux в режиме Enforcing. В `nginx.conf` задано `root /mephi-web;`. Каталогу назначен тип `httpd_sys_content_t`: `semanage fcontext -a -t httpd_sys_content_t "/mephi-web(/.*)?"` и `restorecon -Rv /mephi-web` |

### Раздел 6. Аутентификация

|||
|---|---|
| 6.1 Ограничение входа | Модуль `pam_access` подключён в `/etc/pam.d/login`. В `/etc/security/access.conf` запрещен локальный вход группе `curators`: `-:curators:LOCAL` |
| 6.2 Пароли | Срок действия: `chage -M 90` для всех пользователей. Минимальная длина: `minlen = 12` в `/etc/security/pwquality.conf` |

### Раздел 7. Тестирование

|||
|---|---|
| 7.1 Web-страница | Файл `/mephi-web/index.html` с текстом `Hello from Student: М265854` |
| 7.2 Проверка доступности | `curl http://localhost/` |

### Раздел 8. Публикация

Файлы проекта:

| Файл | Что проверяется |
|---|---|
| `mephi-screenshot.png` | Визуальное подтверждение |
| `history.out` | Выполненные команды |
| `ping.out` | Работоспособность сети |
| `dnf.out` | Управление пакетами |
| `stat.out` | Права доступа и контекст SELinux |
| `journalctl.out` | Запуск веб-сервера |
| `getcap.out` | Настройка привилегий |
| `getenforce.out` | Режим SELinux |
| `curl.out` | Результат тестирования |
| `fstab` | Монтирование файловых систем (`/etc/fstab`) |
| `passwd` | Пользователи (`/etc/passwd`) |
| `shadow` | Пароли пользователей (`/etc/shadow`) |
| `group` | Группы (`/etc/group`) |
| `pwquality.conf` | Настройка пароля (`/etc/security/pwquality.conf`) |

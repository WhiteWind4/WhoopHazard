# Оптимизация загрузки Raspberry Pi для RotorHazard

Проверено на: Raspberry Pi 5, Debian 12 (bookworm), RotorHazard 4.3.0-dev.3
Результат: загрузка с ~1 мин 32 сек → ~9 сек

## 1. Автоперезапуск сервиса при крашах

Проблема: `corrupted size vs. prev_size` (heap corruption в нативных библиотеках ws281x/bme280) — недетерминистический краш, сервис падает и не поднимается.

Добавить в `/lib/systemd/system/rotorhazard.service` секцию `[Service]`:

```ini
Restart=on-failure
RestartSec=3
StartLimitIntervalSec=60
StartLimitBurst=5
```

## 2. Отключить Plymouth (splash-экран)

Экономия: ~1 мин 28 сек

```bash
sudo systemctl mask plymouth-quit-wait.service
```

## 3. Отключить NetworkManager-wait-online

Экономия: ~1 мин (был скрыт за Plymouth)

```bash
sudo systemctl disable NetworkManager-wait-online.service
```

## 4. Уменьшить таймаут DHCP

Экономия: ~25 сек

В `/usr/local/bin/dhcp-or-static.sh` заменить:
```bash
timeout 30 dhclient -v $IFACE
```
на:
```bash
timeout 5 dhclient -v $IFACE
```

Логика сохраняется: пробует DHCP 5 сек, если не получил — ставит статический IP.

## 5. Переключить target на multi-user

Экономия: ~1 мин 23 сек (lightdm ждёт GPU)

```bash
sudo systemctl set-default multi-user.target
```

## После всех изменений

```bash
sudo systemctl daemon-reload
sudo reboot
```

Проверка:
```bash
systemd-analyze                              # общее время загрузки
systemd-analyze blame | head -10             # топ медленных сервисов
systemd-analyze critical-chain rotorhazard   # цепочка зависимостей RH
systemctl status rotorhazard                 # статус сервиса
```

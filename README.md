# corepost-install

`corepost-install` это интерактивный bash-установщик и provisioning-компонент для демо-стенда CorePost.

Цель установщика:
- зарегистрировать устройство на сервере;
- сохранить provisioning bundle локально;
- сгенерировать `/etc/corepost-preboot.conf`;
- установить preboot runtime в `initramfs-tools` и пересобрать initramfs;
- (пока временно) скачать и включить stub `corepost-agent` systemd unit, чтобы в дальнейшем агент заменился без изменения install flow;
- (для проверки на VM) уметь собрать демонстрационный LUKS-носитель и прогнать дешифровку через preboot-скрипт.

## Как это работает

Основной скрипт: `install.sh`.

### 1) Загрузка preboot и agent из GitHub raw

Installer скачивает нужные артефакты по HTTPS из GitHub raw content:
- `corepost-preboot`:
  - `initramfs-tools/hooks/corepost-preboot`
  - `initramfs-tools/scripts/local-top/corepost-preboot`
- `corepost-agent` (stub):
  - `dist/corepost-agent.service`
  - `dist/corepost-agent.sh`

Ссылки на raw-контент параметризованы через env:
- `COREPOST_GITHUB_ORG` (по умолчанию `CorePost`)
- `COREPOST_GITHUB_RAW_HOST` (по умолчанию `raw.githubusercontent.com`)
- `COREPOST_PREBOOT_REPO`, `COREPOST_PREBOOT_REF`
- `COREPOST_AGENT_REPO`, `COREPOST_AGENT_REF`

### 2) Регистрация на сервере

Installer просит у пользователя:
- `COREPOST_SERVER_URL` (base URL сервера)
- `COREPOST_ADMIN_TOKEN` (admin token для `POST /admin/register`)

После успешной регистрации сохраняет полный ответ сервера (provisioning bundle) в
`/var/lib/corepost-install/provisioning.json` с правами `0600`.

### 3) Конфиг preboot

Installer генерирует `/etc/corepost-preboot.conf` (права `0600`), включая:
- `COREPOST_SERVER_URL`
- `COREPOST_DEVICE_ID`
- `COREPOST_DEVICE_SECRET`
- `COREPOST_UNLOCK_PROFILE` (`2fa` или `3fa`)
- параметры сети и ретраев
- `COREPOST_LUKS_DEVICE`, `COREPOST_LUKS_NAME`

### 4) Установка preboot в initramfs

Installer кладёт файлы в:
- `/etc/initramfs-tools/hooks/corepost-preboot`
- `/etc/initramfs-tools/scripts/local-top/corepost-preboot`

и выполняет `update-initramfs -u`.

Hook добавляет в initramfs минимальный набор утилит (`curl`, `openssl`, `cryptsetup`, DHCP-клиент, `ip`) и копирует
в initramfs `/etc/corepost-preboot.conf` и CA bundle.

### 5) 3FA (опционально)

Если пользователь соглашается включить `3fa`, installer пытается найти removable device (например, USB storage),
показывает список кандидатов, и только после явного подтверждения форматирует выбранный девайс в `ext4` с label
`COREPOST_USB`. Preboot-контур проверяет наличие фактора через путь `/dev/disk/by-label/COREPOST_USB`.

### 6) VM demo дешифровки (опционально)

Для демонстрации на VM installer может:
1. запросить `user password` (не сохраняется);
2. создать demo LUKS-носитель с паролем открытия `user_password + unlockToken`;
3. вызвать preboot-скрипт в userspace, чтобы он:
   - проверил состояние на сервере;
   - получил `unlockToken`;
   - выполнил `cryptsetup luksOpen`;
4. проверить, что mapping открыт.

Это даёт воспроизводимую проверку того, что preboot действительно “разрешает/запрещает” доступ к зашифрованному блоку.

## Команды

Установка:

```bash
sudo ./install.sh install
```

Reconfigure (обновить runtime/конфиг без полного удаления):

```bash
sudo ./install.sh reconfigure
```

Что делает `reconfigure`:
- повторно скачивает preboot runtime и agent stub и пересобирает initramfs;
- берёт `COREPOST_SERVER_URL` из env или из текущего `/etc/corepost-preboot.conf` (с возможностью изменить интерактивно);
- по умолчанию сохраняет существующий provisioning bundle и конфиг;
- по запросу пользователя может заново зарегистрировать устройство на сервере (это по сути ротация секретов), сохранить новый bundle и перегенерировать `/etc/corepost-preboot.conf`.

Примечание про agent:
- installer после обновления unit/script/env гарантированно перезапускает `corepost-agent.service`, чтобы systemd перечитал новые файлы и `/etc/corepost-agent.env`.

Удаление установленных артефактов:

```bash
sudo ./install.sh uninstall
```

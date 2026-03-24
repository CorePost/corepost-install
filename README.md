### corepost-install
регистрирует устройство и подготавливает локальную конфигурацию системы.
Скрипт получает provisioning bundle, пишет конфиг preboot, разворачивает agent и обновляет initramfs.
Поддерживается подготовка USB-фактора для профиля 3FA и локальный 2FA.
Компонент работает как root на Linux системе с curl, openssl и initramfs-tools.

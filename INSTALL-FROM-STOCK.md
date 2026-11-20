# Установка OpenWrt с заводской прошивки ASUS RT‑AX53U

Установка RouterVPN с нуля состоит из двух этапов. Сначала устанавливается официальный OpenWrt, затем на чистый OpenWrt устанавливается RouterVPN вместе с его зависимостями.

Эта инструкция по прошивке рассчитана на ASUS RT‑AX53U и ASUS RT‑AX1800U с платформой `ramips/mt7621`. Проверьте полное название модели на наклейке перед загрузкой образа. [OpenWrt Firmware Selector для этой модели](https://firmware-selector.openwrt.org/?id=asus_rt-ax53u&target=ramips%2Fmt7621).

## 1. Подготовьте два образа OpenWrt

| Этап | Файл |
| --- | --- |
| Переход из веб-интерфейса ASUS | `openwrt-ramips-mt7621-asus_rt-ax53u-initramfs-factory.trx` из официального каталога [OpenWrt Snapshot](https://downloads.openwrt.org/snapshots/targets/ramips/mt7621/). |
| Постоянная установка из OpenWrt | `openwrt-25.12.5-ramips-mt7621-asus_rt-ax53u-squashfs-sysupgrade.bin` из [OpenWrt 25.12.5](https://downloads.openwrt.org/releases/25.12.5/targets/ramips/mt7621/). |

Загрузите оба файла на ПК и сверьте SHA‑256. В PowerShell используйте `Get-FileHash -Algorithm SHA256 <путь-к-файлу>`. Переходная сборка Snapshot меняется, поэтому сверяйте её с `sha256sums` из того же каталога в день загрузки. SHA‑256 указанного стабильного образа — `2407f2154c04bdb70fe98c92a50067410c44484875515f7346446b5338c1b1f4`. [Контрольные суммы Snapshot](https://downloads.openwrt.org/snapshots/targets/ramips/mt7621/sha256sums) · [Контрольные суммы 25.12.5](https://downloads.openwrt.org/releases/25.12.5/targets/ramips/mt7621/sha256sums).

## 2. Загрузите переходный образ через ASUS

1. Сохраните заводские настройки и официальный образ ASUS для возврата. Подключите ПК кабелем к LAN‑порту именно того роутера, который прошиваете. [Загрузки ASUS RT‑AX53U](https://www.asus.com/supportonly/rt-ax53u/helpdesk_bios/).
2. В заводской панели ASUS откройте **Administration → Firmware Upgrade** и загрузите `initramfs-factory.trx` для этой модели.
3. Дождитесь завершения записи и перезагрузки, сохраняя питание роутера. После загрузки OpenWrt подключитесь по адресу `192.168.1.1`. Если в переходном образе отсутствует веб-интерфейс, используйте `ssh root@192.168.1.1`. [Описание переходного образа в OpenWrt](https://github.com/openwrt/openwrt/commit/9e9dc4f7c23de613f06d34cba45378e48b1b0c99) · [SSH в OpenWrt](https://openwrt.org/docs/guide-quick-start/sshadministration).

Если ASUS отклоняет TRX, остановитесь и перепроверьте модель, образ и его хеш. Для этой модели OpenWrt также документирует U‑Boot TFTP через последовательную консоль с отдельным образом `squashfs-factory.bin`. [Инструкция U‑Boot TFTP для RT‑AX53U](https://github.com/openwrt/openwrt/commit/8c00fd9b4519bf0ef8fb3470a6df421b9f38c03c).

## 3. Установите стабильный OpenWrt 25.12.5

Убедитесь, что загрузился OpenWrt на нужном роутере. Команда `ubus call system board` должна показать модель RT‑AX53U и платформу `ramips/mt7621`. Скопируйте проверенный образ `squashfs-sysupgrade.bin` с ПК. [Обновление OpenWrt через LuCI и SSH](https://openwrt.org/docs/guide-user/installation/generic.sysupgrade).

```powershell
scp -O .\openwrt-25.12.5-ramips-mt7621-asus_rt-ax53u-squashfs-sysupgrade.bin root@192.168.1.1:/tmp/openwrt-25.12.5.bin
```

В SSH на роутере сначала проверьте образ. При успешной проверке команда ниже установит систему с заводскими настройками OpenWrt и перезагрузит роутер. [Параметры sysupgrade](https://openwrt.org/docs/techref/sysupgrade).

```sh
printf '%s\n' '2407f2154c04bdb70fe98c92a50067410c44484875515f7346446b5338c1b1f4  /tmp/openwrt-25.12.5.bin' | sha256sum -c - &&
sysupgrade -T /tmp/openwrt-25.12.5.bin &&
sysupgrade -n /tmp/openwrt-25.12.5.bin
```

Запись начнётся только после успешной проверки SHA‑256 и совместимости. После загрузки откройте `http://192.168.1.1`, задайте пароль `root` и подтвердите версию `25.12.5` командой `ubus call system board`. [Первый пароль OpenWrt](https://openwrt.org/faq/initial_root_password).

## 4. Подготовьте установку RouterVPN

Кандидат RouterVPN 3.28.1 RC1 предназначен для ASUS RT‑AX53U с чистым OpenWrt `25.12.5`, платформой `ramips/mt7621` и ядром `6.12.94` с ABI `35fb64ea09ee15c2f267bd5436b7f178`. В ZIP входят панель, Xray, офлайн-пакеты и настройки для подписанных обновлений. [Чистый установщик 3.28.1 RC1](https://github.com/barabbaslatro/RouterVPN-Panel-Releases/releases/tag/installer-v3.28.1-rc1).

1. Скачайте ZIP, его `.sha256` и `.sig`. Сверьте SHA‑256 и проверьте подпись ZIP публичным ключом `panel-update.pub.pem`, затем распакуйте все файлы в одну папку. Команда проверки есть в `README-RU.md` внутри архива. [Публичный ключ в выпуске панели 3.28.1](https://github.com/barabbaslatro/RouterVPN-Panel-Releases/releases/tag/v3.28.1).
2. Подключите Windows-ПК кабелем к LAN роутера, запустите `Start-RouterVPN.cmd` и выберите **Check**. Проверка сверит модель, версию системы, зависимости и доступное место.
3. Для пробной установки на отдельном роутере с чистым OpenWrt после успешной проверки выберите **Clean install**. Укажите страну Wi‑Fi и задайте отдельные пароли для Wi‑Fi и панели. Дальнейшие шаги описаны в `README-RU.md`.

Режим **Check** и тесты отката прошли на рабочем OpenWrt. Полная установка на отдельном устройстве с чистым OpenWrt пока предстоит. На роутере с установленной панелью обновления доступны в разделе **Настройки → Обновление панели**.


# Обновления панели RouterVPN

Этот репозиторий хранит выпуски панели RouterVPN для OpenWrt. Версия **3.28.0** добавляет ручную проверку обновлений в **Настройки → Обновление панели**. Первый выпуск для этого обновлятора — **3.28.1**.

## Файлы выпуска v3.28.1

| Файл | Назначение |
| --- | --- |
| `routervpn-bootstrap-3.28.0.tar.gz` | Первая установка обновлятора по SSH на проверенную базу `3.27.0-manual-performance-rc1`. |
| `panel-update.pub.pem` | Публичный ключ RSA‑3072. На роутере хранится в `/etc/routervpn/panel-update.pub.pem`. |
| `routervpn-update.json` | Манифест с версией, адресом, размером и SHA‑256 пакета. |
| `routervpn-update.sig` | Подпись манифеста. |
| `routervpn-panel-3.28.1.json` | Пакет с файлами панели. |

Приватный ключ хранится у выпускающего и нужен для подписи следующих версий.

## Первая установка

Сохраните резервную копию файлов панели. На роутере должны быть `curl`, корневые TLS-сертификаты, `sha256sum`, `ucode` с модулями `fs` и `math` и `openssl` из пакета `openssl-util`. Перенесите архив и публичный ключ в `/tmp`, затем выполните от root.

```sh
tar -xzpf /tmp/routervpn-bootstrap-3.28.0.tar.gz -C /
cp /tmp/panel-update.pub.pem /etc/routervpn/panel-update.pub.pem
chown root:root /etc/routervpn/panel-update.pub.pem
chmod 0644 /etc/routervpn/panel-update.pub.pem
/etc/init.d/routervpn-panel-update enable
```

## Источник обновлений

Роутер получает подписанный выпуск через публичный GitHub Releases. Для своего HTTPS-зеркала выпустите манифест с адресом пакета на зеркале и подпишите его тем же ключом.

Для обновления с GitHub создайте `/etc/routervpn/panel-update.json` со следующим содержимым и правами `0600`.

```json
{
  "manifestUrl": "https://github.com/barabbaslatro/RouterVPN-Panel-Releases/releases/latest/download/routervpn-update.json"
}
```

Панель скачивает пакет после подтверждения владельца, проверяет подпись и контрольную сумму. При сбое записи восстанавливает прежние файлы. Включённая служба восстанавливает незавершённую установку при следующей загрузке OpenWrt.

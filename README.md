# 🐶 BobiVPN Checker

Автоматический чекер VPN ключей на **Go + Xray-core**.  
Проверяет каждый ключ через **реальный трафик** — запускает xray, поднимает локальный SOCKS5 и гоняет тесты связности, смены IP и скорости.

---

## ✨ Возможности

- **100% реальная проверка** — ключ тестируется как настоящий пользователь через xray-core
- **Поддержка протоколов**: VLESS, VMess, Trojan, Shadowsocks
- **Умная сортировка**: RU → Европа → Азия → Unknown в конце
- **Несколько форматов вывода**: обычный список, base64, формат Happ/v2rayNG, JSON-отчёт
- **Разбивка по странам**: отдельный файл для каждой страны
- **Lite-версия**: только RU, DE, FR, FI, Прибалтика
- **GitHub Actions**: автозапуск каждые 3 часа

---

## 🗂 Структура проекта

```
BobiVpn/
├── cmd/checker/main.go          # Точка входа
├── internal/
│   ├── checker/                 # Логика проверки (5 этапов)
│   ├── config/                  # Настройки и флаги стран
│   ├── geo/                     # GeoIP через ip-api.com
│   ├── keys/                    # Парсинг VPN URI
│   ├── output/                  # Генерация файлов результатов
│   ├── subscription/            # Загрузка подписок
│   └── xray/                    # Управление процессами xray
├── subscriptions.txt            # Список URL подписок
├── vpn-checker                  # Скомпилированный бинарник (Linux)
└── .github/workflows/           # GitHub Actions
```

---

## 🔍 Как работает проверка (5 этапов)

```
1. TCP Ping       → сервер доступен + измеряем latency
2. Xray Start     → запускаем xray с SOCKS5 на локальном порту
3. Connectivity   → HTTP-запрос через прокси (google.com/generate_204)
4. IP Check       → сравниваем exit-IP с нашим (IP ДОЛЖЕН смениться)
5. Download       → замеряем скорость (качество соединения)
```

Ключ считается **рабочим** только если IP изменился — иначе трафик не идёт через прокси.

---

## 🚀 Запуск локально

### Требования
- Go 1.26+
- [xray-core](https://github.com/XTLS/Xray-core/releases) — положить `xray.exe` (Windows) или `xray` (Linux) рядом

### Windows
```powershell
# Скачать xray-core → https://github.com/XTLS/Xray-core/releases
# Распаковать xray.exe в папку проекта

go run ./cmd/checker/ -xray ./xray.exe -subs subscriptions.txt
```

### Linux
```bash
go run ./cmd/checker/ -xray ./xray -subs subscriptions.txt
```

### Параметры
```
-xray           Путь к бинарнику xray (default: "xray")
-subs           Файл со списком подписок (default: "subscriptions.txt")
-j              Параллельных проверок (default: 50)
-tcp-timeout    TCP таймаут в сек (default: 5)
-proxy-timeout  Прокси таймаут в сек (default: 20)
-start-delay    Ожидание запуска xray в сек (default: 2)
-max-latency    Максимальный пинг мс (default: 3000)
```

---

## 📂 Выходные файлы

| Файл | Содержимое |
|------|-----------|
| `vpn.txt` | Рабочие ключи (оригинальные) |
| `vpn_base64.txt` | То же в base64 |
| `vpn_renamed.txt` | С красивыми именами `🇷🇺 Russia \| Yandex 1` |
| `vpn_renamed_base64.txt` | То же в base64 |
| `bobi_vpn.txt` | Формат Happ/v2rayNG с заголовком |
| `bobi_vpn_base64.txt` | То же в base64 |
| `bobi_vpn_lite.txt` | Только RU + Европа (Lite) |
| `vpn_report.json` | Детальный отчёт: страны, провайдеры, пинги |
| `countries/` | Отдельный файл для каждой страны |

---

## ⚙️ GitHub Actions

Автоматически запускается **каждые 3 часа**.  
Результаты коммитятся обратно в репозиторий.

```yaml
# .github/workflows/vpn-checker.yml
schedule:
  - cron: '0 21,0,3,6,9,12,15,18 * * *'
```

### Подписки
Добавляй URL в `subscriptions.txt` (по одному на строку):
```
# Комментарии поддерживаются
https://example.com/subscription
https://другой-сервер.com/keys
```

---

## 🔧 Сборка бинарника

```bash
# Linux (для GitHub Actions)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags="-s -w" -o vpn-checker ./cmd/checker/

# Windows
go build -o vpn-checker.exe ./cmd/checker/
```

---

## 📦 Поддерживаемые протоколы

| Протокол | Поддержка |
|----------|-----------|
| VLESS | ✅ TCP, WS, gRPC, XHTTP + TLS/Reality |
| VMess | ✅ TCP, WS, gRPC, HTTP/2 |
| Trojan | ✅ TCP, WS, gRPC + TLS |
| Shadowsocks | ✅ Все cipher |
| Hysteria2 | ❌ Не поддерживается xray-core |

---

<div align="center">
  <b>🐶 BobiVPN — только проверенные серверы</b>
</div>

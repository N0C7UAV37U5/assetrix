# Домены и HTTPS: настройка

- **Дата:** 2026-11-26
- **Решения:** D7 (HTTPS через DNS-01), D17 (схема URL), D18 (роли доменов), D19 (развёртывание и доступ телефонов) в [`decisions_log.md`](decisions_log.md)
- **Задачи:** Backlog 17, 30

## Домены

| Домен | Роль | Что заводим сейчас |
|---|---|---|
| `flowerp.app` | Рабочие инстансы продукта | `ut.flowerp.app` → рабочий сервер |
| `flowerp.org` | Сайт, документация, сообщество | Пока ничего |
| `mycorium.org` | Экосистема + лаборатория | `*.lab.mycorium.org` → хоумлаб |

Все три — у Porkbun, авто-продление включено.

## Как это работает без VPS

- **DNS:** публичная A-запись указывает на **локальный** IP сервера. Устройства во внутренней сети по этому имени попадают на сервер, снаружи адрес недостижим. Входящие подключения из интернета не нужны.
- **Сертификат (DNS-01):** Caddy через API Porkbun создаёт TXT-запись `_acme-challenge`, Let's Encrypt проверяет её в публичном DNS и выдаёт сертификат, Caddy запись удаляет. Продление — само, примерно раз в 60 дней. Все соединения исходящие.
- **Видно публично:** внутренний IP (в DNS) и имя хоста (в журналах Certificate Transparency). Риск низкий.

## Состояние (2026-11-27)

| Что | Статус |
|---|---|
| `flowerp.app`: API Access, API-ключ | Готово |
| Рабочий сервер: исходящий HTTPS к `api.porkbun.com` и `acme-v02.api.letsencrypt.org` | Проверено, прокси не нужен |
| Рабочий сервер: порты 80/443 | Свободны (стандартный Apache из установки Debian отключён) |
| A-запись `ut.flowerp.app`, заглушка `*` удалена | Готово; DNS предприятия отдаёт внутренний IP |
| Телефон в гостевом Wi-Fi | Имя резолвится, сервер недоступен (отдельная VLAN) — так и задумано; на работе тестируем с ПК и веб-камеры, телефон — в хоумлабе (D19) |
| `mycorium.org`: API Access, `*.lab` | Следующий шаг, для тестов с телефона в хоумлабе |

---

## Шаг 1. Porkbun: API-ключ (делает автор)

1. Porkbun → **Account → API Access** → создать ключ.
2. Получится пара: **API key** (`pk1_…`) и **Secret API key** (`sk1_…`).
3. Сохранить в менеджер паролей. В чат, в репозиторий и в issue не отправлять.

## Шаг 2. Porkbun: включить API для доменов (делает автор)

В **Details** каждого нужного домена включить переключатель **API Access**:
- `flowerp.app` — да;
- `mycorium.org` — да;
- `flowerp.org` — пока не нужно.

Без этого переключателя DNS-01 не пройдёт даже с правильным ключом.

## Шаг 3. Porkbun: DNS-записи

| Домен | Тип | Host | Значение | Комментарий |
|---|---|---|---|---|
| `flowerp.app` | A | `ut` | IP рабочего сервера (локальный) | Рабочий пилот |
| `mycorium.org` | A | `*.lab` | IP хоумлаба (локальный) | Все тестовые инстансы |
| `flowerp.app`, `mycorium.org` | CAA | (корень) | `0 issue "letsencrypt.org"` | Необязательно: сертификаты выдаёт только Let's Encrypt |

В интерфейсе: **Account → Domain Management** → строка домена → **Details** → **DNS**. В поле Host — только `ut` (окончание `.flowerp.app` Porkbun добавит сам), TTL по умолчанию.

Для сервисов не из FlowERP на том же сервере (Grafana, Portainer, старые веб-интерфейсы) — одна запись `*.ut` → тот же IP; дальше новые имена (`grafana.ut.flowerp.app`) добавляются только в Caddyfile.

Если Porkbun добавил записи-заглушки парковки (ALIAS корня и CNAME `*` на `pixie.porkbun.com`), их можно удалить на `flowerp.app`, чтобы произвольные имена не резолвились в парковку. Сайт на корне появится позже.

## Шаг 4. Проверка DNS

С рабочего ПК и с телефона в рабочем Wi-Fi:

```powershell
nslookup ut.flowerp.app
```

```powershell
nslookup test.lab.mycorium.org
```

Должны вернуться локальные IP. Если телефон в рабочем Wi-Fi имя не разрешает — DNS этой сети блокирует публичные имена с частными IP (DNS rebinding protection), см. D7.

Проверка у DNS-серверов Porkbun напрямую (без кешей): `nslookup ut.flowerp.app curitiba.ns.porkbun.com`. Если в ответе `canonical name = pixie.porkbun.com` — своей записи нет, срабатывает заглушка `*` (например, запись добавлена не тому домену).

Диагностика с телефона: `ERR_NAME_NOT_RESOLVED` — проблема DNS; `ERR_CONNECTION_REFUSED` — сервер доступен, порт закрыт; `ERR_CONNECTION_TIMED_OUT` — пакеты не доходят (другая VLAN или файрвол). Домен `.app` в HSTS preload: браузер всегда идёт на `https://`, порт 443.

## Шаг 5. Сервер: подготовка, ключи и Caddy

### Подготовка сервера

1. Исходящий доступ — оба запроса должны вернуть заголовки (404 у Porkbun нормально: у API нет страницы в корне):
   ```
   curl -sI https://api.porkbun.com
   curl -sI https://acme-v02.api.letsencrypt.org/directory
   ```
2. Порты 80 и 443 свободны:
   ```
   sudo ss -tlnp | grep -E ':80|:443'
   ```
   Debian может поставить Apache при установке (tasksel «web server»). Если он не нужен: `sudo systemctl disable --now apache2`. Отдельный веб-сервер не нужен и потом: статику (`file_server`), PHP (`php_fastcgi`), контейнеры и другие машины сети (`reverse_proxy`) обслуживает Caddy.

### Ключи

На каждом сервере ключи лежат в `.env` рядом с `docker-compose.yml`. Файл в `.gitignore`, права `600`:

```
PORKBUN_API_KEY=pk1_...
PORKBUN_API_SECRET_KEY=sk1_...
```

- Формат `KEY=значение`: без пробелов вокруг `=`, без кавычек и `export`.
- Окончания строк только **LF**: с CRLF к ключу прилипает `\r`, и Porkbun отвечает ошибкой авторизации. Надёжнее создать файл на сервере (`nano .env`).
- `.env` сам в контейнер не попадает — у сервиса `caddy` нужен `env_file: .env`.

### Маршрутизация по контейнерам

DNS указывает на сервер, а не на контейнер. Caddy — единственная точка входа: только он публикует порты 80/443, остальные контейнеры без `ports:`, в общей Docker-сети, доступны Caddy по имени сервиса.

```yaml
services:
  caddy:
    build: ./caddy
    env_file: .env
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data          # сертификаты, не терять
    networks: [web]

  assetrix:
    image: assetrix:latest
    networks: [web]               # без ports: доступен только через Caddy

networks:
  web:
volumes:
  caddy_data:
```

Модули FlowERP — по пути (`/assetrix`, `/paginator`, D17); сторонние приложения, которые плохо работают под префиксом, — по отдельному имени (`grafana.ut.flowerp.app`).

### Caddy

Caddy собирается с модулем Porkbun (версии закрепляются при создании каркаса, Backlog 6):

```dockerfile
FROM caddy:2-builder AS builder
RUN xcaddy build --with github.com/caddy-dns/porkbun

FROM caddy:2
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

Caddyfile рабочего сервера (набросок; имена параметров модуля сверить с его README):

```
{
    acme_dns porkbun {
        api_key {env.PORKBUN_API_KEY}
        api_secret_key {env.PORKBUN_API_SECRET_KEY}
    }
}

ut.flowerp.app {
    redir /a/* /assetrix{uri} 302
    handle_path /assetrix/* {
        reverse_proxy assetrix:8000
    }
}
```

На хоумлабе — блок `*.lab.mycorium.org` с маршрутизацией по имени хоста:

```
*.lab.mycorium.org {
    @acme host acme.lab.mycorium.org
    handle @acme {
        redir /a/* /assetrix{uri} 302
        handle_path /assetrix/* {
            reverse_proxy assetrix-acme:8000
        }
    }
}
```

**Сертификаты:** каждый сервер получает только свои имена — рабочий `ut.flowerp.app`, хоумлаб `*.lab.mycorium.org` (D18).

## Шаг 6. Проверка HTTPS

1. С телефона открыть `https://ut.flowerp.app/assetrix/` — сертификат валиден, без предупреждений.
2. Камера в сканере запрашивает разрешение и работает.

---

## Чек-лист

- [x] API-ключ создан и сохранён в менеджере паролей
- [x] API Access включён для `flowerp.app`
- [ ] API Access включён для `mycorium.org`
- [x] Рабочий сервер: исходящий HTTPS к Porkbun и Let's Encrypt
- [x] Рабочий сервер: порты 80/443 свободны
- [x] Запись `ut.flowerp.app` создана, заглушка `*` удалена
- [x] DNS `ut.flowerp.app` проверен с ПК в сети предприятия
- [ ] Запись `*.lab.mycorium.org` создана, DNS проверен с телефона в домашней сети
- [ ] `.env` с ключами на серверах (права `600`)
- [ ] Caddy получил сертификаты, телефон открывает HTTPS без предупреждений

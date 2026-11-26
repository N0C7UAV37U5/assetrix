# Домены и HTTPS: настройка

- **Дата:** 2026-11-26
- **Решения:** D7 (HTTPS через DNS-01), D17 (схема URL), D18 (роли доменов) в [`decisions_log.md`](decisions_log.md)
- **Задачи:** Backlog 17, 30

## Домены

| Домен | Роль | Что заводим сейчас |
|---|---|---|
| `flowerp.app` | Рабочие инстансы продукта | `ut.flowerp.app` → рабочий сервер |
| `flowerp.org` | Сайт, документация, сообщество | Пока ничего |
| `mycorium.org` | Экосистема + лаборатория | `*.lab.mycorium.org` → хоумлаб |

Все три — у Porkbun, авто-продление включено.

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

## Шаг 5. Сервер: ключи и Caddy

На каждом сервере ключи лежат в `.env` рядом с `docker-compose.yml`. Файл в `.gitignore`, права `600`:

```
PORKBUN_API_KEY=pk1_...
PORKBUN_API_SECRET_KEY=sk1_...
```

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

- [ ] API-ключ создан и сохранён в менеджере паролей
- [ ] API Access включён для `flowerp.app` и `mycorium.org`
- [ ] Записи `ut.flowerp.app` и `*.lab.mycorium.org` созданы
- [ ] DNS проверен с ПК и с телефона в рабочем Wi-Fi
- [ ] `.env` с ключами на серверах (права `600`)
- [ ] Caddy получил сертификаты, телефон открывает HTTPS без предупреждений

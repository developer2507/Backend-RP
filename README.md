# FastAPI Admin (Dark)

## Подготовка БД
Создай БД `fastapi_admin` в PostgreSQL:
```sql
CREATE DATABASE fastapi_admin;
```

## Переменные окружения (.env)
Все конфигурации и секреты вынесены в `Backend/.env`.

1. Скопируй шаблон:
```bash
cp .env.example .env
```
2. Заполни `.env` своими значениями (БД, ROOT, AzeriCard, Firebase).

## Запуск (Windows / Linux)
```bash
python -m venv .venv
# Windows:
. .venv/Scripts/activate
# Linux/macOS:
# source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Открой http://127.0.0.1:8000/login

## Запуск через Docker
Из папки `Application/Backend`:

```bash
docker compose up --build
```

После старта:
- Backend: http://127.0.0.1:8000/login
- Healthcheck: http://127.0.0.1:8000/healthz

Важно для Docker:
- `docker-compose.yml` поднимает только backend-контейнер (без PostgreSQL).
- PostgreSQL должна быть запущена отдельно (локально, на сервере или в другом compose).
- В `.env` укажи параметры внешней БД (`PG_HOST`, `PG_PORT`, `PG_DB`, `PG_USER`, `PG_PASSWORD`).
- Если PostgreSQL работает на хост-машине и backend запущен в Docker, для Windows/macOS обычно нужен `PG_HOST=host.docker.internal`.

Вход ROOT берётся из `.env`:
- `ROOT_USERNAME`
- `ROOT_PASSWORD`

## Политика ролей
- ROOT: полный доступ, может создавать/редактировать/удалять всех, видеть/сбрасывать временные пароли.
- ADMIN: может управлять только OPERATOR и RESIDENT. Не может редактировать/удалять ADMIN/ROOT.
- OPERATOR, RESIDENT: без прав управления пользователями.

## Особенности
- Для новых пользователей генерируется временный пароль (видно на странице пользователeй).
- При первом входе (или после сброса) — принудительная смена пароля.
- Для root смена не требуется по умолчанию.

## AzeriCard (MPI / 3-D Secure)

- Официальная документация: [developer.azericard.com](https://developer.azericard.com/) (прод; часто нужна авторизация в браузере).
- Тестовая / песочница: [developer-test.azericard.com](https://developer-test.azericard.com/).
- Шлюз MPI (из руководств AzeriCard): **test** — `https://testmpi.3dsecure.az/cgi-bin/cgi_link`, **prod** — `https://mpi.3dsecure.az/cgi-bin/cgi_link`. В `.env` поля `AZERICARD_GATEWAY_URL` и `AZERICARD_API_URL` должны совпадать с режимом терминала.

## AzeriCard Apple Pay / Google Pay
- Для отдельного wallet-терминала заполните в `.env`:
  - `AZERICARD_TERMINAL_WALLET`
  - `AZERICARD_PRIVATE_KEY_WALLET`
  - `AZERICARD_PUBLIC_KEY_WALLET`
- `AZERICARD_CALLBACK_URL` должен быть публичным full URL (не `localhost`) и используется как `BACKREF`.
- `POST /api/azericard/initiate` принимает дополнительные поля для wallet-потока:
  - `wallet_provider`: `google_pay` или `apple_pay`
  - `wallet_token`: токен кошелька (для Google Pay передается как `GPAYTOKEN`)
  - `wallet_eci`, `wallet_tavv`: 3DS-поля для Apple Pay (если требуются вашим сценарием)
- Конфиг Google Pay для фронта читается из `GET /api/azericard/wallet-config`.
- Для Google Pay обязательно заполните:
  - `AZERICARD_GPAY_GATEWAY` (обычно `azericardgpay`)
  - `AZERICARD_GPAY_GATEWAY_MERCHANT_ID` (выдаётся Azericard; без него Google Pay выдаёт OR_BIBED_06)
- Дополнительно:
  - `AZERICARD_GPAY_ENVIRONMENT` (`TEST` или `PRODUCTION`)
  - `AZERICARD_GPAY_MERCHANT_ID` (обычно обязателен в PRODUCTION)
  - `AZERICARD_GPAY_MERCHANT_NAME`
- Никогда не передавайте партнёрам private key. Для подключения выдаётся только public key (RSA 2048) и callback URL.

# beauty-telegram-bot

Гибкий Telegram-бот для салонов красоты и отдельных мастеров: запись клиентов, управление услугами и товарами, напоминания, многоуровневая админка. Проект запускается в Docker и масштабируется под десятки и сотни инстансов без конфликтов, хранит записи в Google Sheets.

## Ключевые фишки
- Простая установка: одна команда bash (wget | bash) с вашим TELEGRAM_BOT_TOKEN и OWNER_ID
- Из коробки Docker + docker-compose: изоляция и масштабируемость для множества салонов/мастеров
- Клиентские сценарии: запись/отмена, связь с мастером, заявки на продукцию
- Админ-панель: управление мастерами, продукцией, рассылки, блокировки
- Панель мастера: видит свои записи, пишет клиентам, управляет услугами и товарами
- Напоминания: за 2 дня и за 4 часа до визита
- Анти-флуд: лимит 2 записи в день на клиента, защита от массового бронирования
- Хранилище: Google Sheets (клиенты, визиты, услуги, мастера, заявки и покупки)
- Масштабируемая архитектура и понятная структура проекта

## Установка за 1 команду

На сервере с Docker и docker-compose:

```bash
bash -c "$(wget -qO- https://raw.githubusercontent.com/Mortyo666/beauty-telegram-bot/main/docker/init.sh)" \
  -- \
  --token "YOUR_TELEGRAM_BOT_TOKEN" \
  --owner "YOUR_TELEGRAM_USER_ID" \
  --sheet "YOUR_GOOGLE_SHEET_ID"
```

Скрипт:
- создаст .env
- развернёт docker-compose с сервисом бота
- выполнит первичную инициализацию Google Sheets

## Структура проекта (план)

- bot.py — точка входа бота (python-telegram-bot/aiogram), роутинг команд и хэндлеров
- config/
  - settings.py — чтение .env, конфигурация токенов/лимитов/таймингов
  - logging.py — логирование
- google_sheets/
  - client.py — авторизация (service account), обёртка над Google Sheets API
  - models.py — схемы таблиц (клиенты, визиты, мастера, услуги, товары, заявки)
  - repository.py — CRUD-операции
  - init_templates/ — шаблоны таблиц и диапазонов
- docker/
  - Dockerfile — образ бота (slim + non-root)
  - docker-compose.yml — сервисы bot + (опционально) scheduler/worker
  - init.sh — инсталлятор (wget | bash), генерация .env, запуск compose
- admin/
  - routes_admin.py — команды и меню администратора
  - routes_master.py — меню мастера
  - acl.py — роли, доступы, блок-лист
- features/
  - booking/ — запись, отмена, слоты, напоминания, лимиты
  - products/ — товары, заявки, подтверждение покупки
  - messaging/ — рассылки, сообщения клиентам, связь с мастером
  - anti_flood/ — защита от спама, капча по необходимости
- ui/
  - keyboards.py — инлайн/реплай клавиатуры
  - texts.py — тексты и локализация (RU, EN опционально)
- scripts/
  - migrate_sheets.py — инициализация/миграции структуры Google Sheets
  - seed_demo.py — демо-данные (пример салона/мастера/услуг)
- README.md — документация

## Google Sheets: схемы таблиц (план)
- masters: id, last_name, first_name, role, tg_username, active
- services: id, master_id, name, price, duration_min, active
- bookings: id, client_id, master_id, service_id, date, time, status, created_at, product_id(optional)
- clients: id, tg_id, username, name, phone(optional), blocked
- products: id, name, price, stock(optional), active
- product_requests: id, client_id, product_id, status, comment, created_at

## Базовые команды
- /start — приветствие и меню
- /book — запись к мастеру
- /my — мои записи, отмена
- /products — каталог и заявки
- /admin — панель администратора (доступ только для админов)
- /master — панель мастера

## Масштабирование
- Каждый бот — отдельный контейнер с собственным .env и Google Sheet
- Дополнительно: фоновые воркеры для рассылок и напоминаний (Celery/APScheduler)

## Дальнейшие улучшения
- Онлайн-оплата (Stripe/ЮKassa)
- Интеграция с CRM (Amo/Битрикс24)
- Виджеты расписания и WebApp мини-приложение в Telegram

Лицензия: MIT (план). Pull Requests приветствуются.

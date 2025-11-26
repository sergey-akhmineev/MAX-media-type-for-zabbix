# Zabbix → MAX Webhook

Интеграция Zabbix с платформой MAX через Webhook‑media type.  
Скрипт отправляет уведомления о проблемах и их восстановлении в MAX‑чат или конкретному пользователю, с наглядными иконками:

- при проблеме — `❌` (красный крест) и текст триггера;
- при восстановлении — `✅` (зелёная галочка) и текст триггера.

---

## Возможности

- Отправка сообщений из Zabbix в:
  - личный диалог с пользователем (`user_id`);
  - групповой чат (`chat_id`).
- Поддержка форматирования текста (`html` или `markdown`).
- Настройка:
  - отключения превью ссылок;
  - уведомлений / тихих сообщений (`notify`).

---

## Требования

- Zabbix 5.0+ (или любая версия с media type типа **Webhook** и `HttpRequest` в JavaScript).
- Аккаунт и бот на платформе MAX.
- Токен бота MAX:
  - в интерфейсе MAX: `Чат-бот и мини-приложение → Настройка → Токен доступа`. (Не забудьте включить "Добавление в групповые чаты")
---

## Установка

### 1. Создать media type в Zabbix

1. Откройте: `Administration → Media types → Create media type`.
2. Заполните:
   - **Name**: `MAX`
   - **Type**: `Webhook`
   - **Status**: `Enabled`

3. Во вкладке **Script** вставьте код из файла Script
### 2. Параметры media type

Во вкладке **Parameters** добавьте параметры (Name → Value):

Обязательные:

- `Token`  
  Значение: токен бота MAX  
  Пример: `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

Один из двух (выберите схему, которую будете использовать):

- Вариант 1: отправка **конкретному пользователю**:
  - `UserId` → `{ALERT.SENDTO}`  
    В поле "Send to" у пользователя будет `user_id` из MAX.

- Вариант 2: отправка **в чат**:
  - `ChatId` → `{ALERT.SENDTO}`  
    В поле "Send to" у пользователя будет `chat_id` из MAX.

Текст:

- `Subject` → `{ALERT.SUBJECT}`
- `Message` → `{ALERT.MESSAGE}`

Опциональные:

- `Format` → `html`  
  или `markdown` (если хотите использовать форматирование MAX)

- `DisableLinkPreview` → `1` или `0`  
  `1` — отключить превью ссылок, `0` — включить

- `Notify` → `1` или `0`  
  `1` — присылать уведомления участникам чата, `0` — тихие сообщения

---

### 3. Настройка пользователя (Media)

1. Откройте: `Administration → Users`.
2. Выберите пользователя, перейдите на вкладку **Media**.
3. Добавьте запись:
   - **Type**: `MAX`
   - **Send to**: 1234567890
- для ChatId‑варианта: укажите `chat_id` чата MAX, например: -4567456745674576
Важно: без двоеточий, пробелов и прочих символов, только число.
   - Остальные поля по вашему стандарту (severity, period и т.д.)
4. Для запроса id чата надо, добавить бота в чат и выполнить запрос : curl -k -X GET "https://platform-api.max.ru/chats"  -H "Authorization: {token}", вернется что то вроде:
   - {"chats":[{"chat_id":-4567456745674576,"type":"chat","status":"active","title":"Тестовый чат","last_event_time":1763965283164,"participants_count":3,"is_public":false,"messages_count":3}]}
   
## Шаблоны сообщений Zabbix

# Problem  
• Message type: Problem  
Subject:  
❌ Problem: {EVENT.NAME}  
Message:  
❌ Проблема началась в {EVENT.TIME} {EVENT.DATE}  
Проблема: {EVENT.NAME}  
Хост: {HOST.NAME}  
Важность: {EVENT.SEVERITY}  
Данные: {EVENT.OPDATA}  
ID проблемы: {EVENT.ID}  
{TRIGGER.URL}  

# Problem recovery  
• Message type: Problem recovery  
Subject:  
✅ Resolved: {EVENT.RECOVERY.NAME}  
Message:  
✅ Проблема решена в {EVENT.RECOVERY.TIME} {EVENT.RECOVERY.DATE}  
Проблема: {EVENT.RECOVERY.NAME}  
Длительность: {EVENT.DURATION}  
Хост: {HOST.NAME}  
Важность: {EVENT.SEVERITY}  
ID проблемы: {EVENT.ID}  
{TRIGGER.URL}  

# Problem update (по желанию)  
Subject:  
ℹ️ Updated: {EVENT.NAME}  
Message:  
{USER.FULLNAME} {EVENT.UPDATE.ACTION} проблему в {EVENT.UPDATE.DATE} {EVENT.UPDATE.TIME}.  
{EVENT.UPDATE.MESSAGE}  
Текущий статус: {EVENT.STATUS}, возраст: {EVENT.AGE}, подтверждена: {EVENT.ACK.STATUS}.  


<img width="722" height="775" alt="image" src="https://github.com/user-attachments/assets/75ade0e2-9a49-4089-a864-616e6f8cecdf" />

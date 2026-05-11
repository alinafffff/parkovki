# 5. Интерфейсы (ICD-lite)

В данном разделе описаны внешние интерфейсы системы парковок ParkPoint, обеспечивающие взаимодействие клиентского приложения с серверной частью. Контракты включают REST API и событийные сообщения.

---

## 1. Авторизация пользователя

| Параметр | Значение |
|----------|----------|
| Endpoint | POST /api/v1/auth/login |
| Версия | v1 |
| Тип | REST API |

### Входные данные (request body)
- email (string) — email пользователя  
- password (string) — пароль  

### Выходные данные
- userId (long) — идентификатор пользователя  
- role (string) — роль пользователя  
- token (string) — JWT-токен  

### Ошибки
- 401 — неверный логин или пароль  
- 403 — пользователь заблокирован  
- 500 — ошибка сервера  

---

## 2. Создание бронирования парковочного места

| Параметр | Значение |
|----------|----------|
| Endpoint | POST /api/v1/booking |
| Версия | v1 |
| Тип | REST API |

### Входные данные
- idClient (long) — пользователь  
- idParkingSpace (long) — парковочное место  
- startTime (timestamp) — начало бронирования  
- endTime (timestamp) — конец бронирования  

### Выходные данные
- bookingId (long)  
- status (string) — CONFIRMED / REJECTED  

### Ошибки
- 409 — место уже забронировано  
- 400 — некорректные данные  
- 500 — ошибка сервера  

---

## 3. Получение списка парковочных зон

| Параметр | Значение |
|----------|----------|
| Endpoint | GET /api/v1/parking-zones |
| Версия | v1 |
| Тип | REST API |

### Параметры запроса
- latitude (double) — координаты  
- longitude (double) — координаты  
- radius (int) — радиус поиска  

### Ответ
- zones[]:
  - id (long)
  - title (string)
  - address (string)
  - availableSpaces (int)

### Ошибки
- 400 — неверные параметры  
- 500 — ошибка сервера  

---

## 4. Событие: создание бронирования

| Параметр | Значение |
|----------|----------|
| Event | booking.created |
| Версия | v1 |
| Тип | Kafka event |

### Поля события
- eventId (string) — уникальный ID события  
- bookingId (long)  
- userId (long)  
- parkingSpaceId (long)  
- createdAt (timestamp)  
- status (string)  

### Ошибки обработки
- дубликат eventId (идемпотентность)
- ошибка сериализации события

---

## 5. Событие: сообщение в чате

| Параметр | Значение |
|----------|----------|
| Event | chat.message.sent |
| Версия | v1 |
| Тип | Kafka event |

### Поля события
- messageId (long)  
- chatId (long)  
- senderId (long)  
- recipientId (long)  
- content (text)  
- timestamp (timestamp)  

### Ошибки
- message too large  
- invalid chatId  
- serialization error  

---

## 6. Событие: жалоба пользователя

| Параметр | Значение |
|----------|----------|
| Event | complaint.created |
| Версия | v1 |
| Тип | Kafka event |

### Поля события
- complaintId (long)  
- complainantId (long)  
- accusedId (long)  
- text (text)  
- status (string)  
- createdAt (timestamp)  

### Ошибки
- empty complaint text  
- invalid user reference  
- system error  

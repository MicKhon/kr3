# FastAPI Security Suite — KR3 
Полноценная система безопасности для серверных приложений: аутентификация, авторизация, JWT, RBAC, защита документации.
## Быстрый старт
### Скачать проект
```bash
git clone https://github.com/MicKhon/kr3 && cd kr3
Подготовить окружение
python -m venv .venv && source .venv/bin/activate # Linux/Mac
или
.venv\Scripts\activate # Windows
```
### Установить пакеты
```bash
pip install -r requirements.txt
Активировать конфигурацию
cp .env.example .env # Linux/Mac
или
copy .env.example .env # Windows
```
### Запустить сервер
```bash
uvicorn main:app --host 127.0.0.1 --port 8000 --reload
Приложение доступно: http://127.0.0.1:8000
```
## Архитектура решения
```bash
kr3/
├── main.py # Точки входа, маршруты, зависимости
├── models.py # Pydantic-схемы и структуры данных
├── database.py # Работа с SQLite, миграции
├── security.py # Хеширование, JWT, валидация
├── limiter.py # Rate-limiting логика
├── requirements.txt # Зависимости проекта
├── .env.example # Шаблон конфигурации
├── .gitignore # Исключения для контроля версий
└── README.md # Этот файл
```
## Модули безопасности
Базовая аутентификация (HTTP Basic)
Проверка доступа:
```bash
curl -u admin:secret123 http://localhost:8000/auth/basic
Ошибка авторизации вернёт 401 + WWW-Authenticate заголовок:
```
```bash
curl -u fake:wrong http://localhost:8000/auth/basic
Регистрация + хеширование паролей
```
### Создать аккаунт:
```bash
curl -X POST http://localhost:8000/users/register
-H "Content-Type: application/json"
-d '{"username":"dev_user","password":"Str0ngP@ss!"}'
```
### Авторизоваться:
```bash
curl -u dev_user:Str0ngP@ss! http://localhost:8000/users/me
Пароли хранятся как bcrypt-хеши. Сравнение строк — через secrets.compare_digest().
JWT-сессии
```
### Получить токен:
```bash
curl -X POST http://localhost:8000/auth/jwt
-H "Content-Type: application/json"
-d '{"username":"dev_user","password":"Str0ngP@ss!"}'
Ответ:
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...","token_type":"bearer"}
Использовать токен:
```
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
http://localhost:8000/secure/data
Ограничение запросов (Rate Limiting)
Эндпоинт /users/register: лимит 1 запрос в минуту
Эндпоинт /auth/jwt: лимит 5 запросов в минуту
При превышении лимита возвращается 429 Too Many Requests
Ролевая модель (RBAC)
```
### Роли и доступ:
```bash
guest: Чтение публичных ресурсов
user: Чтение + обновление своих данных
admin: Полный CRUD, управление пользователями
Примеры запросов:
Зона для авторизованных пользователей:

curl -H "Authorization: Bearer USER_TOKEN" http://localhost:8000/workspace
Админ-панель (только admin):
curl -H "Authorization: Bearer ADMIN_TOKEN" http://localhost:8000/admin/dashboard
Попытка доступа без прав вернёт 403 Forbidden
CRUD: Управление задачами (Todo)
Создать задачу:
curl -X POST http://localhost:8000/tasks
-H "Content-Type: application/json"
-d '{"title":"Сделать ДЗ","description":"KR3 по FastAPI"}'
Ответ:
{"id":1,"title":"Сделать ДЗ","description":"KR3 по FastAPI","completed":false}
Получить все задачи:
curl http://localhost:8000/tasks
Получить одну задачу:
curl http://localhost:8000/tasks/1
Обновить задачу:
curl -X PUT http://localhost:8000/tasks/1
-H "Content-Type: application/json"
-d '{"completed":true}'
Удалить задачу:
curl -X DELETE http://localhost:8000/tasks/1
Данные хранятся в SQLite (control_work.db). Схема создаётся автоматически при первом запуске.
Конфигурация окружения
Файл .env (не коммитить!):
Режим: DEV / PROD
APP_MODE=DEV
Защита документации (только DEV)
DOCS_LOGIN=superuser
DOCS_PASS=docs_secure_2026
JWT параметры
JWT_KEY=your_ultra_secret_key_here_change_in_prod
JWT_ALGO=HS256
JWT_TTL_MINUTES=30
База данных
DB_PATH=sqlite:///./control_work.db
Поведение по режимам:
DEV: /docs защищён Basic Auth, /redoc скрыт, /openapi.json защищён Basic Auth
PROD: /docs, /redoc, /openapi.json возвращают 404
Проверка работоспособности
Сервер запустился?
curl http://localhost:8000/health
Ответ: {"status":"ok"}
Регистрация работает?
curl -s -X POST http://localhost:8000/users/register
-H "Content-Type: application/json"
-d '{"username":"test","password":"test123"}' | jq
JWT выдаётся?
TOKEN=$(curl -s -X POST http://localhost:8000/auth/jwt
-H "Content-Type: application/json"
-d '{"username":"test","password":"test123"}' | jq -r .access_token)
Защищённый ресурс доступен с токеном?
curl -s -H "Authorization: Bearer $TOKEN" http://localhost:8000/secure/data
Зависимости
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
pydantic>=2.5.0
pydantic-settings>=2.1.0
passlib[bcrypt]>=1.7.4
python-jose[cryptography]>=3.3.0
slowapi>=0.1.9
python-dotenv>=1.0.0
Установка:
pip install -r requirements.txt
Git-workflow для сдачи
Инициализация (если ещё не сделано)
git init
git add .
git commit -m "feat: complete KR3 security implementation"
Привязка к удалённому репозиторию
git remote add origin https://github.com/MicKhon/kr3.git
git branch -M main
Отправка (используйте PAT если включён 2FA)
git push -u origin main
Ссылка для сдачи в СДО:
https://github.com/MicKhon/kr3
Устранение неполадок
Problem: ModuleNotFoundError
Solution: Проверьте активацию venv и pip install -r requirements.txt
Problem: 401 при доступе к /docs
Solution: В режиме DEV укажите DOCS_LOGIN/DOCS_PASS из .env
```
Problem: Токен не принимается
Solution: Убедитесь, что в заголовке Authorization: Bearer <token> (с пробелом)
Problem: База не создаётся
Solution: Проверьте права на запись в папку проекта
Problem: Лимит запросов срабатывает рано
Solution: В limiter.py можно временно увеличить пороги для тестов

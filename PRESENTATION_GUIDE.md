# Task Manager - Ръководство за Презентация

## 📋 Общ Преглед на Проекта

**Task Manager** е пълнофункционално web приложение за управление на задачи с календарен интерфейс. Приложението е изградено като microservices архитектура с Docker Compose, включваща frontend, backend API, база данни и reverse proxy с SSL.

---

## 🏗️ Архитектура и Технологии

### Технологичен Стек

**Frontend:**
- React 19 (модерен UI framework)
- Vite (бърз build tool)
- Tailwind CSS (utility-first CSS framework)
- Custom Modal компоненти

**Backend:**
- Node.js 20
- Express.js (RESTful API)
- MySQL2 (database driver)

**Infrastructure:**
- Docker & Docker Compose (containerization)
- MySQL 8.0 (релационна база данни)
- Nginx (reverse proxy, load balancer)
- Self-signed SSL сертификати (HTTPS)

**Monitoring:**
- Docker Stats (resource monitoring)

---

## 📝 Развитие на Проекта - Стъпка по Стъпка

### Стъпка 1: База Данни (MySQL)

**Какво направихме:**
- Настроихме MySQL 8.0 контейнер с Docker Compose
- Създадохме database schema с таблица `tasks`
- Конфигурирахме health checks за надеждност
- Настроихме persistent storage (volumes)

**Database Schema:**
```sql
CREATE TABLE tasks (
  id INT AUTO_INCREMENT PRIMARY KEY,
  task_date DATE NOT NULL,
  title VARCHAR(255) NOT NULL,
  notes TEXT NULL,
  is_done BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Ключови точки:**
- Автоматично timestamp управление
- Boolean flag за статус на задачата

---

### Стъпка 2: Backend API (Node.js/Express)

**Какво направихме:**
- RESTful API с Express.js
- Connection pool към MySQL
- Пълна CRUD функционалност
- Error handling и валидация

**API Endpoints:**

1. **GET /health** - Health check
   - Проверява статуса на API и базата данни

2. **GET /tasks?month=YYYY-MM** - Списък с задачи
   - Опционален филтър по месец
   - Сортиране по дата

3. **POST /tasks** - Създаване на задача
   - Изисква: `task_date`, `title`
   - Опционално: `notes`

4. **PUT /tasks/:id** - Редактиране на задача
   - Поддържа частично обновяване (patch)
   - Може да обновява: `title`, `notes`, `is_done`

5. **DELETE /tasks/:id** - Изтриване на задача

**Технически детайли:**
- CORS enabled за cross-origin requests
- JSON body parsing
- Prepared statements (SQL injection protection)
- Proper HTTP status codes

---

### Стъпка 3: Frontend (React)

**Какво направихме:**
- React приложение с Vite
- Календарен интерфейс с визуализация на задачи
- Tailwind CSS за стилизация
- Custom Modal компоненти за UX

**Ключови Функционалности:**

1. **Календарен Грид:**
   - Винаги показва 6 седмици (42 клетки)
   - Дни от предишен/следващ месец са видими, но по-светли
   - Фиксирана височина (без "jumping")
   - Визуален индикатор за брой задачи на ден

2. **CRUD Операции:**
   - **Create:** Форма за добавяне на задача
   - **Read:** Автоматично зареждане на задачи за текущия месец
   - **Update:** Модален прозорец за редактиране
   - **Delete:** Модален прозорец за потвърждение

3. **UX Подобрения:**
   - Заменени browser-native `prompt()` и `confirm()` с custom модали
   - Responsive дизайн
   - Hover ефекти и transitions
   - Error handling и feedback

**Технически Детайли:**
- `useMemo` за оптимизация на изчисления
- `useEffect` за data fetching
- State management с React hooks
- API abstraction layer (`api.js`)

---

### Стъпка 4: Docker Containerization

**Какво направихме:**
- Dockerfile за всеки service
- Docker Compose за orchestration
- Environment variables за конфигурация
- Volume management за persistent data

**Docker Services:**

1. **db (MySQL):**
   - Health checks
   - Persistent storage
   - Auto-initialization с SQL scripts

2. **api (Backend):**
   - Multi-stage build (оптимизация)
   - Production dependencies only
   - Environment-based configuration

3. **frontend (React):**
   - Build process в контейнера
   - Static file serving
   - Production-optimized build

4. **nginx (Reverse Proxy):**
   - Request routing
   - SSL termination
   - HTTP to HTTPS redirect

---

### Стъпка 5: Nginx Reverse Proxy

**Какво направихме:**
- Настроихме Nginx като reverse proxy
- Конфигурирахме маршрутизация:
  - `/` → Frontend (port 3000)
  - `/api/*` → Backend API (port 3001)
- HTTP to HTTPS redirect
- Load balancing готовност

**Защо Nginx?**
- Единен входен пункт (single entry point)
- SSL termination
- Може да балансира натоварването
- Оптимизация на статични файлове

---

### Стъпка 6: SSL/TLS (HTTPS)

**Какво направихме:**
- Генерирахме self-signed SSL сертификат
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout nginx/ssl/server.key \
     -out nginx/ssl/server.crt
   ```
- Конфигурирахме Nginx за HTTPS
- HTTP автоматично пренасочва към HTTPS

**Защо SSL?**
- Криптирана комуникация
- Security best practice
- Подготовка за production (с валиден сертификат)

**Ограничения на self-signed:**
- Браузър показва предупреждение (нормално)
- Не е подходящ за production (трябва Let's Encrypt или платен сертификат)

---

### Стъпка 7: Monitoring (Docker Stats)

**Какво направихме:**
- Настроихме Docker Stats за мониторинг
- Документирахме използването
- Скрипт за лесно стартиране

**Какво показва:**
- CPU usage
- Memory usage
- Network I/O
- Block I/O

**Защо е важно:**
- Откриване на performance проблеми
- Resource planning
- Debugging на issues

---

## 🎯 Ключови Технически Решения

### 1. Календар с Фиксирана Височина

**Проблем:** Календарът "прескачаше" при смяна на месеци заради различния брой седмици.

**Решение:**
- Винаги рендираме точно 42 клетки (6 седмици × 7 дни)
- Дни от предишен/следващ месец са видими, но по-светли
- CSS Grid с фиксирани редове: `gridTemplateRows: "repeat(6, minmax(60px, 1fr))"`

### 2. API Endpoint Configuration

**Проблем:** Frontend трябва да работи и локално, и в production.

**Решение:**
- Environment variable `VITE_API_BASE`
- Fallback към `/api` (работи с Nginx)
- Подготовка за GitHub Pages deployment

### 3. Modal Components

**Проблем:** Browser-native `prompt()` и `confirm()` не са модерни.

**Решение:**
- Custom `Modal` компонент
- Reusable за edit и delete
- По-добър UX и контрол

### 4. Date Handling

**Проблем:** Различни формати между MySQL (ISO string) и frontend (YYYY-MM-DD).

**Решение:**
- Парсване на ISO string в Date object
- Консистентно форматиране в `dateKey()` функция
- Правилна конверсия в `tasksByDate` memoization

---

## 📊 Функционални Изисквания (Изпълнени)

✅ **Минимални условия:**
- Потребителски интерфейс за CRUD операции
- База данни за съхранение
- Backend API

✅ **Общ вид:**
- Docker Compose решение
- Load balancer (Nginx)
- Frontend (React)
- Backend (Node.js/Express)
- База данни (MySQL)
- Self-signed SSL сертификати
- Monitoring (Docker Stats)

---

## 🚀 Deployment

### Локално Стартиране

```bash
# 1. Създаване на .env файл
cat > .env << EOF
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=calendar_db
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass
EOF

# 2. Стартиране на всички услуги
docker compose up -d --build

# 3. Достъп
# https://localhost (или http://localhost - автоматично пренасочва)
```

### GitHub Pages (Frontend Only)

- Автоматично deployment с GitHub Actions
- Frontend се build-ва и deploy-ва при push
- Backend трябва да е деплойнат отделно

---

## 📁 Структура на Проекта

```
calendar_app/
├── backend/              # Node.js/Express API
│   ├── src/
│   │   ├── index.js     # API endpoints
│   │   └── db.js        # Database connection pool
│   ├── Dockerfile
│   └── package.json
├── frontend/             # React приложение
│   ├── src/
│   │   ├── App.jsx      # Главен компонент (календар)
│   │   ├── Modal.jsx    # Reusable modal компонент
│   │   ├── api.js       # API функции
│   │   └── index.css    # Tailwind CSS
│   ├── Dockerfile
│   └── package.json
├── mysql/
│   └── init/
│       └── 01_schema.sql # Database schema
├── nginx/
│   ├── nginx.conf        # Nginx конфигурация
│   └── ssl/              # SSL сертификати
├── .github/
│   └── workflows/
│       └── deploy.yml     # GitHub Actions workflow
├── docker-compose.yml    # Docker Compose конфигурация
├── .env                  # Environment variables (не се commit-ва)
└── README.md             # Документация
```

---

## 🎨 UI/UX Особености

1. **Визуална Иерархия:**
   - Текущият месец е с пълен цвят
   - Предишен/следващ месец са по-светли (faded)
   - Избраният ден е подчертан

2. **Интерактивност:**
   - Hover ефекти на клетките
   - Badge с брой задачи на ден
   - Smooth transitions

3. **Модали:**
   - Overlay за фокус
   - Click-outside-to-close
   - Keyboard-friendly (ESC за затваряне)

---

## 🔒 Security Features

1. **SQL Injection Protection:**
   - Prepared statements във всички queries

2. **HTTPS:**
   - Криптирана комуникация
   - SSL/TLS termination в Nginx

3. **Environment Variables:**
   - Чувствителни данни не се commit-ват
   - `.env` файл в `.gitignore`

4. **CORS:**
   - Конфигуриран за контролиран достъп

---

## 📈 Performance Оптимизации

1. **Frontend:**
   - `useMemo` за изчисления на tasksByDate
   - Оптимизиран re-rendering
   - Production build с minification

2. **Backend:**
   - Connection pooling
   - Database indexes
   - Efficient SQL queries

3. **Docker:**
   - Multi-stage builds
   - Production-only dependencies
   - Layer caching

---

## 🐛 Проблеми и Решения

### Проблем 1: Tailwind CSS v4 Compatibility
**Решение:** Downgrade към v3.4.19 с правилна PostCSS конфигурация

### Проблем 2: Circular Dependency в CSS
**Решение:** Премахнато `@apply grid`, използвани директни CSS properties

### Проблем 3: Tasks не се показват след създаване
**Решение:** Коригирана date conversion от ISO string към YYYY-MM-DD

### Проблем 4: Календар "прескача"
**Решение:** Фиксирани 42 клетки с правилна логика за предишен/следващ месец

---

## 🎓 Какво Научихме

1. **Docker & Docker Compose:**
   - Multi-container applications
   - Service orchestration
   - Health checks и dependencies

2. **Microservices Architecture:**
   - Разделение на frontend, backend, database
   - Reverse proxy за маршрутизация
   - Service communication

3. **Modern Web Development:**
   - React hooks и state management
   - RESTful API design
   - Responsive UI с Tailwind CSS

4. **DevOps Practices:**
   - Containerization
   - SSL/TLS setup
   - Monitoring
   - CI/CD с GitHub Actions

---

## 🔮 Бъдещи Подобрения

- [ ] User authentication и authorization
- [ ] Real-time updates (WebSockets)
- [ ] Task categories и tags
- [ ] Export/Import функционалност
- [ ] Mobile app (React Native)
- [ ] Advanced monitoring (Netdata, Prometheus)
- [ ] Automated testing (Jest, Cypress)
- [ ] Production-ready SSL (Let's Encrypt)

---

## 📚 Полезни Команди

```bash
# Стартиране
docker compose up -d --build

# Спиране
docker compose down

# Логове
docker compose logs -f [service_name]

# Monitoring
docker stats

# Health check
curl -k https://localhost/api/health

# Database access
docker exec -it calendar-mysql mysql -u appuser -p calendar_db
```

---

## 💡 Ключови Takeaways

1. **Docker Compose** прави лесно управлението на multi-service приложения
2. **Nginx** е мощен инструмент за reverse proxy и SSL termination
3. **React hooks** правят state management по-чист и по-лесен
4. **Tailwind CSS** ускорява UI development значително
5. **Proper error handling** е критично за user experience
6. **Monitoring** помага да се открият проблеми рано

---

*Този документ е подготвен за помощ при създаване на презентация за Task Manager проекта.*





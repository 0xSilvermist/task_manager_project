# Task Manager 

## Описание

Приложение за създаване и управление на задачи. Можеш да добавяш задачи да различни дни, да маркираш задачите като изпълнени, да ги редактираш и триеш.

## Архитектура
Soon 

## Компоненти

### 1. Frontend (React)
- **Технология:** React + Vite + Tailwind CSS

### 2. Backend API (Node.js/Express)
- **Технология:** Node.js + Express

### 3. Database (MySQL)
- **Технология:** MySQL 8.0
- **База данни:** calendar_db
- **Таблица:** tasks


### 5. SSL/TLS
- **Тип:** Self-signed сертификат

## Стартиране

### Първоначална настройка

1. **Клониране/Изтегляне на проекта**
```bash
cd calendar_app
```

2. **Създаване на .env файл**
```bash
cat > .env << EOF
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=calendar_db
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass
EOF
```

3. **Стартиране на контейнерите**
```bash
docker compose up -d --build
```

4. **Проверка на статуса**
```bash
docker compose ps
```

https://localhost (или http://localhost - автоматично пренасочва към HTTPS)

## Деплойване

### GitHub Pages (Frontend Only)

**Важно:** GitHub Pages може да хостира само frontend (статични файлове). Backend и базата данни трябва да са деплойнати отделно.

#### Автоматично деплойване

1. **Активирайте GitHub Pages:**
   - Отидете в Settings → Pages
   - Source: "GitHub Actions"

2. **Настройте Backend API URL (опционално):**
   - Отидете в Settings → Secrets and variables → Actions
   - Добавете нов secret: `VITE_API_BASE`
   - Стойност: URL на вашия деплойнат backend (напр. `https://your-backend.railway.app/api`)
   - Ако не зададете secret, frontend ще използва относителен път `/api` (работи само ако backend е на същия домейн)

3. **Push към main branch:**
   ```bash
   git push origin main
   ```
   GitHub Actions автоматично ще build-не и deploy-не frontend-а.

4. **Достъп:**
   - Вашият сайт ще бъде на: `https://0xSilvermist.github.io/task_manager/`

#### Ръчно деплойване

```bash
cd frontend
npm install
npm run build
# Качете съдържанието на frontend/dist/ в GitHub Pages
```

### Пълно деплойване (Frontend + Backend + Database)

За пълно деплойване на всички компоненти, използвайте платформи, които поддържат Docker:

- **Railway** (препоръчително): https://railway.app
- **Render**: https://render.com
- **Fly.io**: https://fly.io
- **DigitalOcean App Platform**: https://www.digitalocean.com/products/app-platform

## Структура на проекта

```
calendar_app/
├── backend/             # Node.js/Express API
│   ├── src/
│   │   ├── index.js     # API endpoints
│   │   └── db.js        # Database connection
│   └── Dockerfile
├── frontend/            # React приложение
│   ├── src/
│   │   ├── App.jsx      # Главен компонент
│   │   ├── api.js       # API функции
│   │   └── Modal.jsx    # Модални прозорци
│   └── Dockerfile
├── mysql/
│   └── init/
│       └── 01_schema.sql # Database schema
├── nginx/
│   ├── nginx.conf        # Nginx конфигурация
│   └── ssl/              # SSL сертификати
│       ├── server.crt
│       └── server.key
├── docker-compose.yml    # Docker Compose конфигурация
├── .env                  # Environment variables
└── README.md            # Документация
```


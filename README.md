
**Стек:** HTML, CSS, React (frontend), Node.js (backend), REST API, реляционная модель базы данных (наподобие PostgreSQL)

---

## 📖 Введение

Современные веб-приложения требуют комплексного подхода к организации интерфейса, логики взаимодействия клиента с сервером и работы с базой данных. В рамках данной работы была реализована информационная система, предоставляющая функциональность авторизации, регистрации, отображения профиля пользователя и дальнейшей интеграции обучающих материалов и тестов.

Проект разделен на **клиентскую часть** (интерфейс пользователя) и **серверную часть** (обработка логики и данных). Особое внимание уделено структурности HTML-кода, адаптивной верстке, пользовательскому опыту и расширяемости архитектуры проекта.

Также проект включает в себя работу с **реляционной моделью базы данных**, аналогичной PostgreSQL: таблицы, связи, поиск по ключам, хеширование паролей и фильтрация данных по параметрам.

---

## 🧩 Структура проекта

### 1. Интерфейс пользователя (HTML + CSS + React)

#### 📄 Главная страница (`/main`)

Цель — создать интуитивно понятное приветственное окно с кнопками входа и регистрации.

**HTML-разметка (упрощенно):**

```html
<div class="main-container">
  <div class="left-section">
    <img src="/logo.png" alt="Логотип" />
    <h1>Добро пожаловать в систему</h1>
    <p>Обучение, развитие, рост.</p>
    <div class="buttons">
      <a href="/login" class="btn btn-red">Войти</a>
      <a href="/register" class="btn btn-white">Зарегистрироваться</a>
    </div>
  </div>
  <div class="right-section">
    <img src="/welcome.jpg" alt="Welcome Image" />
  </div>
</div>
```

**CSS (фрагмент):**

```css
.main-container {
  display: flex;
  height: 100vh;
  background: linear-gradient(to right, #fff, #fce4ec);
}
.left-section {
  flex: 1;
  padding: 40px;
}
.right-section {
  flex: 1;
  background-position: center;
  background-size: cover;
}
.btn-red {
  background-color: #e53935;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
}
```

#### 📥 Страница входа (`/login`)

Форма входа содержит поля для почты и пароля. После отправки выполняется проверка учетных данных.

```html
<form class="login-form">
  <h2>Вход в систему</h2>
  <label>Email:</label>
  <input type="email" required />
  <label>Пароль:</label>
  <input type="password" required />
  <button type="submit" class="btn-red">Войти</button>
  <a href="/register">Нет аккаунта?</a>
</form>
```

#### 🧾 Страница регистрации (`/register`)

Регистрирует нового пользователя, отправляя данные на сервер.

```html
<form class="register-form">
  <h2>Создание аккаунта</h2>
  <input type="text" placeholder="Имя" required />
  <input type="email" placeholder="Email" required />
  <input type="password" placeholder="Пароль" required />
  <button type="submit" class="btn-red">Зарегистрироваться</button>
</form>
```

---

### 2. Серверная часть (Node.js + REST API)

#### 🌐 API-роутинг

Организация серверных маршрутов осуществляется через `Express.js`. Примеры маршрутов:

```js
// POST /api/register
router.post("/register", async (req, res) => {
  const { name, email, password } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = { name, email, password: hashedPassword };
  await db.insert("users", newUser);
  res.json({ userId: newUser.id });
});

// POST /api/login
router.post("/login", async (req, res) => {
  const user = await db.findByEmail(req.body.email);
  const match = await bcrypt.compare(req.body.password, user.password);
  if (match) res.json({ userId: user.id });
  else res.status(401).json({ error: "Неверные данные" });
});
```

#### 🧠 Хранение данных (модель реляционной БД)

Структура базы данных подобен **PostgreSQL**, где каждый пользователь — это строка в таблице `users`.

**Таблица пользователей:**

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE,
  password TEXT,
  role VARCHAR(50),
  rating INTEGER DEFAULT 0
);
```

**Особенности:**

* Уникальный email
* Пароль хранится в зашифрованном виде (bcrypt)
* Возможность масштабирования: можно добавить таблицы `tests`, `results`, `lessons` с внешними ключами

---

## 👤 Страница профиля (`/profile`)

Профиль пользователя отображает основную информацию, полученную с сервера по `userId`.

**Пример кода компонента:**

```js
useEffect(() => {
  fetch("/api/profile", {
    method: "POST",
    body: JSON.stringify({ userId }),
    headers: { "Content-Type": "application/json" }
  })
    .then(res => res.json())
    .then(data => setUser(data));
}, []);
```

**Отображение:**

```html
<div class="profile-card">
  <h2>Профиль</h2>
  <p><strong>Имя:</strong> {{ user.name }}</p>
  <p><strong>Email:</strong> {{ user.email }}</p>
  <p><strong>Рейтинг:</strong> {{ user.rating }}</p>
  <a href="/tests" class="btn-white">Перейти к тестам</a>
</div>
```

---




---

## 🛡️ Безопасность и защита данных

* Все пароли хэшируются перед хранением
* Используется валидация данных на клиенте и сервере
* Защита маршрутов от неавторизованных пользователей (через проверку `userId`)
* Можно внедрить JWT или сессионную авторизацию при необходимости

---


Вот методичка в формате Markdown (MD) с возможностью копирования кода:

```markdown
# Методичка по проекту "Ресторанные бронирования"

## Содержание
1. [Файл стилей (CSS)](#stylescss)
2. [PHP файлы](#php-файлы)
   - [admin.php](#adminphp)
   - [bookings.php](#bookingsphp)
   - [db.php](#dbphp)
   - [header.php](#headerphp)
   - [index.php](#indexphp)
   - [login.php](#loginphp)
   - [logout.php](#logoutphp)
   - [new_booking.php](#new_bookingphp)
   - [register.php](#registerphp)
3. [SQL файл](#restaurant_bookingsql)

---

## styles.css {#stylescss}

```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Roboto', sans-serif;
    background-color: #F9F9F9;
    color: #333;
    line-height: 1.6;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.nav {
    background-color: #2E4A3D;
    padding: 15px 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.nav__list {
    display: flex;
    gap: 20px;
    list-style: none;
}

.nav__link {
    color: #FFFFFF;
    text-decoration: none;
    font-size: 16px;
    font-weight: 500;
    transition: color 0.3s ease;
}

.nav__link:hover {
    color: #D4A017;
}

.heading {
    font-size: 24px;
    font-weight: 700;
    margin: 20px 0;
    color: #2E4A3D;
}

.table {
    width: 100%;
    border-collapse: collapse;
    background-color: #FFFFFF;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.table__head {
    background-color: #2E4A3D;
    color: #FFFFFF;
}

.table__cell {
    padding: 12px;
    border: 1px solid #E0E0E0;
    font-size: 14px;
}

.table__cell--header {
    font-weight: 500;
}

.form {
    display: flex;
    flex-direction: column;
    gap: 15px;
    max-width: 400px;
    margin: 20px 0;
}

.form__label {
    font-size: 14px;
    font-weight: 500;
    color: #2E4A3D;
}

.form__input,
.form__textarea,
.form__select {
    padding: 10px;
    border: 1px solid #E0E0E0;
    border-radius: 4px;
    font-size: 14px;
    width: 100%;
    transition: border-color 0.3s ease;
}

.form__input:focus,
.form__textarea:focus,
.form__select:focus {
    border-color: #D4A017;
    outline: none;
}

.form__error {
    color: #D32F2F;
    font-size: 12px;
    margin-top: 5px;
}

.button {
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
}

.button--primary {
    background-color: #D4A017;
    color: #FFFFFF;
}

.button--primary:hover {
    background-color: #B38E14;
    transform: scale(1.05);
}

.text {
    font-size: 16px;
    margin: 10px 0;
}

.text__link {
    color: #D4A017;
    text-decoration: none;
}

.text__link:hover {
    text-decoration: underline;
}
```

---

## PHP файлы

### admin.php {#adminphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';
session_start();

if (!isset($_SESSION['user']) || $_SESSION['user']['role'] != 'admin') {
    header('Location: index.php');
    exit;
}

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $booking_id = (int)$_POST['booking_id'];
    $status = $_POST['status'];
    if (in_array($status, ['new', 'completed', 'cancelled'])) {
        $stmt = $pdo->prepare("UPDATE bookings SET status = ? WHERE id = ?");
        $stmt->execute([$status, $booking_id]);
        header('Location: admin.php');
        exit;
    }
}

$stmt = $pdo->prepare("
    SELECT b.*, CONCAT(u.first_name, ' ', u.last_name) AS full_name, u.phone, u.email
    FROM bookings b
    JOIN users u ON b.user_id = u.id
    ORDER BY b.created_at DESC
");
$stmt->execute();
$bookings = $stmt->fetchAll();
?>

<div class="container">
    <h2 class="heading">Панель администратора</h2>
    <?php if (empty($bookings)): ?>
        <p class="text">Бронирований нет.</p>
    <?php else: ?>
        <table class="table">
            <thead class="table__head">
                <tr>
                    <th class="table__cell table__cell--header">ФИО</th>
                    <th class="table__cell table__cell--header">Телефон</th>
                    <th class="table__cell table__cell--header">Email</th>
                    <th class="table__cell table__cell--header">Дата</th>
                    <th class="table__cell table__cell--header">Время</th>
                    <th class="table__cell table__cell--header">Гостей</th>
                    <th class="table__cell table__cell--header">Контактный телефон</th>
                    <th class="table__cell table__cell--header">Статус</th>
                    <th class="table__cell table__cell--header">Отзыв</th>
                    <th class="table__cell table__cell--header">Дата создания</th>
                    <th class="table__cell table__cell--header">Действия</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($bookings as $booking): ?>
                    <tr>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['full_name']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['phone']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['email']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['booking_date']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['booking_time']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['guests']); ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['phone']); ?></td>
                        <td class="table__cell">
                            <?php
                            $status = $booking['status'];
                            echo $status == 'new' ? 'Новое' :
                                 ($status == 'confirmed' ? 'Подтверждено' :
                                 ($status == 'cancelled' ? 'Отменено' : 'Посещение состоялось'));
                            ?>
                        </td>
                        <td class="table__cell"><?php echo $booking['feedback'] ? htmlspecialchars($booking['feedback']) : '-'; ?></td>
                        <td class="table__cell"><?php echo htmlspecialchars($booking['created_at']); ?></td>
                        <td class="table__cell">
                            <form class="form" method="POST">
                                <input type="hidden" name="booking_id" value="<?php echo $booking['id']; ?>">
                                <select class="form__select" name="status" onchange="this.form.submit()">
                                    <option value="new" <?php echo $booking['status'] == 'new' ? 'selected' : ''; ?>>Новое</option>
                                    <option value="completed" <?php echo $booking['status'] == 'completed' ? 'selected' : ''; ?>>Посещение состоялось</option>
                                    <option value="cancelled" <?php echo $booking['status'] == 'cancelled' ? 'selected' : ''; ?>>Отменено</option>
                                </select>
                            </form>
                        </td>
                    </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    <?php endif; ?>
</div>
</body>
</html>
```

### bookings.php {#bookingsphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';

session_start();

if (!isset($_SESSION['user'])) {
    header('Location: login.php');
    exit;
}

$user_id = $_SESSION['user']['id'];
$error = '';

if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['feedback'])) {
    $booking_id = (int)$_POST['booking_id'];
    $feedback_text = trim($_POST['feedback_text']);

    if (empty($feedback_text)) {
        $error = "Текст отзыва не может быть пустым";
    } else {
        $stmt = $pdo->prepare("SELECT status, feedback FROM bookings WHERE id = ? AND user_id = ?");
        $stmt->execute([$booking_id, $user_id]);
        $booking = $stmt->fetch();

        if ($booking && $booking['status'] == 'completed' && is_null($booking['feedback'])) {
            $stmt = $pdo->prepare("UPDATE bookings SET feedback = ? WHERE id = ? AND user_id = ?");
            $stmt->execute([$feedback_text, $booking_id, $user_id]);
            header('Location: bookings.php');
            exit;
        } else {
            $error = "Отзыв можно оставить только для завершённых бронирований без отзыва";
        }
    }
}

$stmt = $pdo->prepare("SELECT * FROM bookings WHERE user_id = ? ORDER BY created_at DESC");
$stmt->execute([$user_id]);
$bookings = $stmt->fetchAll();
?>

<div class="container">
    <h2 class="heading">Мои бронирования</h2>
    <?php if ($error): ?>
        <p class="form__error"><?php echo $error; ?></p>
    <?php endif; ?>
    <table class="table">
        <thead class="table__head">
            <tr>
                <th class="table__cell table__cell--header">Дата</th>
                <th class="table__cell table__cell--header">Время</th>
                <th class="table__cell table__cell--header">Гостей</th>
                <th class="table__cell table__cell--header">Телефон</th>
                <th class="table__cell table__cell--header">Статус</th>
                <th class="table__cell table__cell--header">Отзыв</th>
                <th class="table__cell table__cell--header">Дата создания</th>
                <th class="table__cell table__cell--header">Действия</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach ($bookings as $booking): ?>
            <tr>
                <td class="table__cell"><?php echo htmlspecialchars($booking['booking_date']); ?></td>
                <td class="table__cell"><?php echo htmlspecialchars($booking['booking_time']); ?></td>
                <td class="table__cell"><?php echo htmlspecialchars($booking['guests']); ?></td>
                <td class="table__cell"><?php echo htmlspecialchars($booking['phone']); ?></td>
                <td class="table__cell">
                    <?php
                    $status = $booking['status'];
                    if ($status == 'new') echo 'Новое';
                    elseif ($status == 'confirmed') echo 'Подтверждено';
                    elseif ($status == 'cancelled') echo 'Отменено';
                    elseif ($status == 'completed') echo 'Посещение состоялось';
                    ?>
                </td>
                <td class="table__cell"><?php echo $booking['feedback'] ? htmlspecialchars($booking['feedback']) : '-'; ?></td>
                <td class="table__cell"><?php echo htmlspecialchars($booking['created_at']); ?></td>
                <td class="table__cell">
                    <?php if ($booking['status'] == 'completed' && is_null($booking['feedback'])): ?>
                        <form class="form" method="post" action="bookings.php">
                            <input type="hidden" name="booking_id" value="<?php echo $booking['id']; ?>">
                            <textarea class="form__textarea" name="feedback_text" required pattern=".{1,}" title="Отзыв не может быть пустым"></textarea>
                            <button class="button button--primary" type="submit" name="feedback">Оставить отзыв</button>
                        </form>
                    <?php endif; ?>
                </td>
            </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
</div>
</body>
</html>
```

### db.php {#dbphp}

```php
<?php
$host = 'localhost';
$db = 'restaurant_booking';
$user = 'root';
$pass = '';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8mb4", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Ошибка подключения: " . $e->getMessage());
}
?>
```

### header.php {#headerphp}

```php
<?php
session_start();
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Я буду кушац</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>

<header class="header">
    <div class="header__inner container">
        <div class="logo"><a href="index.php">Gusto</a></div>
        
        <button class="nav__toggle" onclick="document.querySelector('.nav__list').classList.toggle('active')">
            ☰
        </button>
        
        <ul class="nav__list">
            <li><a class="nav__link" href="index.php">Главная</a></li>

            <?php if (isset($_SESSION['user'])): ?>
                <li><a class="nav__link" href="bookings.php">Мои бронирования</a></li>
                <li><a class="nav__link" href="new_booking.php">Забронировать</a></li>

                <?php if ($_SESSION['user']['role'] == 'admin'): ?>
                    <li><a class="nav__link" href="admin.php">Админка</a></li>
                <?php endif; ?>

                <li><a class="nav__link" href="logout.php">Выйти</a></li>
            <?php else: ?>
                <li><a class="nav__link" href="login.php">Войти</a></li>
                <li><a class="nav__link" href="register.php">Регистрация</a></li>
            <?php endif; ?>
        </ul>
    </div>
</header>
<script>
  const toggle = document.querySelector('.nav__toggle');
  const navList = document.querySelector('.nav__list');

  toggle.addEventListener('click', () => {
    navList.classList.toggle('active');
  });
</script>

            </body>
```

### index.php {#indexphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';
?>

<div class="container">
    <h2 class="heading">Добро пожаловать в "Я буду кушац"</h2>
    <p class="text">Забронируйте столик в нашем ресторане!</p>
    <?php if (!isset($_SESSION['user'])): ?>
        <p class="text">Пожалуйста, <a class="text__link" href="login.php">войдите</a> или <a class="text__link" href="register.php">зарегистрируйтесь</a>.</p>
    <?php else: ?>
        <p class="text">Перейдите в <a class="text__link" href="new_booking.php">раздел бронирования</a> или просмотрите <a class="text__link" href="bookings.php">ваши бронирования</a>.</p>
    <?php endif; ?>
</div>
</body>
</html>
```

### login.php {#loginphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';

session_start();
$error = '';

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = trim($_POST['username']);
    $password = trim($_POST['password']);

    $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
    $stmt->execute([$username, $password]);
    $user = $stmt->fetch();

    if ($user) {
        $_SESSION['user'] = [
            'id' => $user['id'],
            'username' => $user['username'],
            'role' => $user['role']
        ];
        header('Location: index.php');
        exit();
    } else {
        $error = "Некорректный логин или пароль";
    }
}
?>

<div class="container">
    <h2 class="heading">Авторизация</h2>
    <form class="form" method="post" action="login.php">
        <label class="form__label">Логин (кириллица, ≥6 символов)</label>
        <input class="form__input" type="text" name="username" required>
        <label class="form__label">Пароль (≥6 символов)</label>
        <input class="form__input" type="password" name="password" required>
        <button class="button button--primary" type="submit">Войти</button>
    </form>
    <?php if ($error): ?>
        <p class="form__error"><?php echo $error; ?></p>
    <?php endif; ?>
</div>
</body>
</html>
```

### logout.php {#logoutphp}

```php
<?php
session_start();
session_destroy();
header('Location: index.php');
exit;
?>
```

### new_booking.php {#new_bookingphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';

if (!isset($_SESSION['user'])) {
    header('Location: login.php');
    exit;
}

$errors = [];

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $user_id = $_SESSION['user']['id'];
    $booking_date = $_POST['booking_date'];
    $booking_time = $_POST['booking_time'];
    $guests = (int)$_POST['guests'];
    $phone = trim($_POST['phone']);

    $date_time = DateTime::createFromFormat('Y-m-d H:i', "$booking_date $booking_time");
    if (!$date_time || $date_time < new DateTime()) {
        $errors[] = "Укажите корректную дату и время (не ранее текущего момента)";
    }

    if ($guests < 1 || $guests > 10) {
        $errors[] = "Количество гостей должно быть от 1 до 10";
    }

    if (!preg_match("/^\+7\(\d{3}\)-\d{3}-\d{2}-\d{2}$/", $phone)) {
        $errors[] = "Телефон должен быть в формате +7(XXX)-XXX-XX-XX";
    }

    if (empty($errors)) {
        $stmt = $pdo->prepare("INSERT INTO bookings (user_id, booking_date, booking_time, guests, phone, status) VALUES (?, ?, ?, ?, ?, 'new')");
        $stmt->execute([$user_id, $booking_date, $booking_time, $guests, $phone]);
        header('Location: bookings.php');
        exit;
    }
}
?>

<div class="container">
    <h2 class="heading">Забронировать столик</h2>
    <form class="form" method="post" action="new_booking.php">
        <label class="form__label">Дата</label>
        <input class="form__input" type="date" name="booking_date" required>
        <label class="form__label">Время</label>
        <input class="form__input" type="time" name="booking_time" required>
        <label class="form__label">Количество гостей (1–10)</label>
        <input class="form__input" type="number" name="guests" min="1" max="10" required>
        <label class="form__label">Контактный телефон (+7(XXX)-XXX-XX-XX)</label>
        <input class="form__input" type="text" name="phone" placeholder="+7(XXX)-XXX-XX-XX" required>
        <button class="button button--primary" type="submit">Забронировать</button>
    </form>
    <?php if (!empty($errors)): ?>
        <?php foreach ($errors as $error): ?>
            <p class="form__error"><?php echo $error; ?></p>
        <?php endforeach; ?>
    <?php endif; ?>
</div>
</body>
</html>
```

### register.php {#registerphp}

```php
<?php
include 'include/db.php';
include 'include/header.php';

$errors = [];

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $username = trim($_POST['username']);
    $password = trim($_POST['password']);
    $first_name = trim($_POST['first_name']);
    $last_name = trim($_POST['last_name']);
    $phone = trim($_POST['phone']);
    $email = trim($_POST['email']);
    $role = 'user';

    if (!preg_match("/^[а-яА-ЯёЁ]{6,}$/u", $username)) {
        $errors[] = "Логин должен содержать только кириллицу и быть не короче 6 символов";
    }

    if (strlen($password) < 6) {
        $errors[] = "Пароль должен быть не короче 6 символов";
    }

    if (!preg_match("/^[а-яА-ЯёЁ ]+$/u", $first_name)) {
        $errors[] = "Имя должно содержать только кириллицу";
    }
    if (!preg_match("/^[а-яА-ЯёЁ ]+$/u", $last_name)) {
        $errors[] = "Фамилия должна содержать только кириллицу";
    }

    if (!preg_match("/^\+7\(\d{3}\)-\d{3}-\d{2}-\d{2}$/", $phone)) {
        $errors[] = "Телефон должен быть в формате +7(XXX)-XXX-XX-XX";
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = "Некорректный формат email";
    }

    $stmt = $pdo->prepare("SELECT COUNT(*) FROM users WHERE username = ?");
    $stmt->execute([$username]);
    if ($stmt->fetchColumn() > 0) {
        $errors[] = "Логин уже занят";
    }

    if (empty($errors)) {
        try {
            $stmt = $pdo->prepare("INSERT INTO users (username, password, first_name, last_name, phone, email, role) VALUES (?, ?, ?, ?, ?, ?, ?)");
            $stmt->execute([$username, $password, $first_name, $last_name, $phone, $email, $role]);
            header('Location: login.php');
            exit();
        } catch (Exception $e) {
            $errors[] = "Ошибка регистрации: " . $e->getMessage();
        }
    }
}
?>

<div class="container">
    <h2 class="heading">Регистрация</h2>
    <form class="form" method="post" action="register.php">
        <label class="form__label">Логин (кириллица, ≥6 символов)</label>
        <input class="form__input" type="text" name="username" required>
        <label class="form__label">Пароль (≥6 символов)</label>
        <input class="form__input" type="password" name="password" required>
        <label class="form__label">Имя</label>
        <input class="form__input" type="text" name="first_name" required>
        <label class="form__label">Фамилия</label>
        <input class="form__input" type="text" name="last_name" required>
        <label class="form__label">Телефон (+7(XXX)-XXX-XX-XX)</label>
        <input class="form__input" type="text" name="phone" placeholder="+7(XXX)-XXX-XX-XX" required>
        <label class="form__label">Email</label>
        <input class="form__input" type="email" name="email" required>
        <button class="button button--primary" type="submit">Зарегистрироваться</button>
    </form>
    <?php if (!empty($errors)): ?>
        <?php foreach ($errors as $error): ?>
            <p class="form__error"><?php echo $error; ?></p>
        <?php endforeach; ?>
    <?php endif; ?>
</div>
</body>
</html>
```

---

## restaurant_booking.sql {#restaurant_bookingsql}

```sql
-- phpMyAdmin SQL Dump
-- version 5.2.0
-- https://www.phpmyadmin.net/
--
-- Хост: 127.0.0.1:3306
-- Время создания: Май 29 2025 г., 22:32
-- Версия сервера: 5.7.39-log
-- Версия PHP: 8.1.9

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- База данных: `restaurant_booking`
--

-- --------------------------------------------------------

--
-- Структура таблицы `bookings`
--

CREATE TABLE `bookings` (
  `id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  `booking_date` date NOT NULL,
  `booking_time` time NOT NULL,
  `guests` int(11) NOT NULL,
  `phone` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL,
  `status` enum('new','confirmed','cancelled','completed') COLLATE utf8mb4_unicode_ci DEFAULT 'new',
  `feedback` text COLLATE utf8mb4_unicode_ci,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

--
-- Дамп данных таблицы `bookings`
--

INSERT INTO `bookings` (`id`, `user_id`, `booking_date`, `booking_time`, `guests`, `phone`, `status`, `feedback`, `created_at`) VALUES
(1, 2, '2025-05-30', '23:38:00', 3, '+7(222)-222-22-22', 'completed', NULL, '2025-05-29 18:36:41'),
(2, 2, '2025-05-30', '23:23:00', 2, '+7(222)-222-22-22', 'new', NULL, '2025-05-29 18:54:15'),
(3, 2, '2025-05-12', '12:31:00', 2, '+7(222)-222-22-22', 'new', NULL, '2025-05-29 18:54:29');

-- --------------------------------------------------------

--
-- Структура таблицы `users`
--

CREATE TABLE `users` (
  `id` int(11) NOT NULL,
  `username` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `password` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `first_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `last_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `phone` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL,
  `email` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL,
  `role` enum('user','admin') COLLATE utf8mb4_unicode_ci DEFAULT 'user',
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

--
-- Дамп данных таблицы `users`
--

INSERT INTO `users` (`id`, `username`, `password`, `first_name`, `last_name`, `phone`, `email`, `role`, `created_at`) VALUES
(1, 'admin', 'restauran', 'Admin', 'Adminov', '+7(999)-999-99-99', 'admin@mail.ru', 'admin', '2025-05-29 17:49:04'),
(2, 'Руслан', '1234567', 'Rus', 'Rus', '+7(222)-222-22-22', 'test@mail.ru', 'user', '2025-05-29 17:50:21');

--
-- Индексы сохранённых таблиц
--

--
-- Индексы таблицы `bookings`
--
ALTER TABLE `bookings`
  ADD PRIMARY KEY (`id`),
  ADD KEY `user_id` (`user_id`);

--
-- Индексы таблицы `users`
--
ALTER TABLE `users`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `username` (`username`);

--
-- AUTO_INCREMENT для сохранённых таблиц
--

--
-- AUTO_INCREMENT для таблицы `bookings`
--
ALTER TABLE `bookings`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=4;

--
-- AUTO_INCREMENT для таблицы `users`
--
ALTER TABLE `users`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=5;

--
-- Ограничения внешнего ключа сохраненных таблиц
--

--
-- Ограничения внешнего ключа таблицы `bookings`
--
ALTER TABLE `bookings`
  ADD CONSTRAINT `bookings_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`);
COMMIT;

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
```
```

Эта методичка содержит все файлы проекта в формате Markdown с подсветкой синтаксиса для CSS, PHP и SQL. Вы можете легко копировать код из соответствующих блоков.

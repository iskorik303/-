# Курсач
Сайт:фронтенд (HTML, CSS или Bootstrap) и бэкенд (Laravel или Node.js). Также нужно будет подключить базу данных. Часть данных должна выводиться из кода на сайт, а другая часть — из базы данных
[Code.js](https://github.com/user-attachments/files/26412631/Code.js)
[HTMLPage1.html](https://github.com/user-attachments/files/26412636/HTMLPage1.html)


HTML

[HTMLPage1.html](https://github.com/user-attachments/files/26412718/HTMLPage1.html)
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Данные: ввод и база данных</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background-color: #f9f9f9;
        }

        h1, h2 {
            color: #333;
        }

        .form-container {
            background: #fff;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin: 10px 0 5px;
            font-weight: bold;
        }

        input, textarea {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }

        button {
            background-color: #28a745;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

            button:hover {
                background-color: #218838;
            }

        .item {
            background: #fff;
            padding: 10px 15px;
            margin-bottom: 10px;
            border-left: 4px solid #28a745;
            border-radius: 4px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }

            .item strong {
                font-size: 1.1em;
            }

            .item p {
                margin: 5px 0 0;
                color: #555;
            }

        .loading {
            text-align: center;
            color: #777;
        }

        .error {
            color: red;
            background: #ffe6e6;
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <h1>Управление записями</h1>

    <div class="form-container">
        <h2>Добавить новую запись</h2>
        <form id="addItemForm">
            <label for="name">Название *</label>
            <input type="text" id="name" name="name" required>
            <label for="description">Описание</label>
            <textarea id="description" name="description" rows="3"></textarea>
            <button type="submit">Добавить</button>
        </form>
    </div>

    <div class="list-container">
        <h2>Список записей </h2>
        <div id="itemsList">
            <div class="loading">Загрузка...</div>
        </div>
    </div>

    <script>
        const itemsListDiv = document.getElementById('itemsList');
        const form = document.getElementById('addItemForm');
        async function fetchItems() {
            try {
                const response = await fetch('/api/items');
                if (!response.ok) throw new Error('Ошибка загрузки данных');
                const items = await response.json();
                renderItems(items);
            } catch (error) {
                itemsListDiv.innerHTML = <div class="error">Не удалось загрузить записи: ${error.message}</div>;
            }
        }
        function renderItems(items) {
            if (!items.length) {
                itemsListDiv.innerHTML = '<div class="loading">Пока нет записей. Добавьте первую!</div>';
                return;
            }
            itemsListDiv.innerHTML = items.map(item => `
                    <div class="item">
                        <strong>${escapeHtml(item.name)}</strong>
                        ${item.description ? <p>${escapeHtml(item.description)}</p> : ''}
                    </div>
                `).join('');
        }
        function escapeHtml(str) {
            if (!str) return '';
            return str.replace(/[&<>]/g, function (m) {
                if (m === '&') return '&amp;'; if (m === '<') return '&lt;';
                if (m === '>') return '&gt;';
                return m;
            }).replace(/[\uD800-\uDBFF][\uDC00-\uDFFF]/g, function (c) {
                return c;
            });
        }
        form.addEventListener('submit', async (e) => {
            e.preventDefault();
            const nameInput = document.getElementById('name');
            const descInput = document.getElementById('description');
            const name = nameInput.value.trim();
            const description = descInput.value.trim();

            if (!name) {
                alert('Название обязательно для заполнения');
                return;
            }

            try {
                const response = await fetch('/api/items', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ name, description })
                });
                if (!response.ok) throw new Error('Ошибка при добавлении');
                nameInput.value = '';
                descInput.value = '';
                fetchItems();
            } catch (error) {
                alert('Не удалось добавить запись: ' + error.message);
            }
        });

        fetchItems();
    </script>
</body>
</html>


JS

[Code.js](https://github.com/user-attachments/files/26412719/Code.js)
/* JavaScript source code*/
import express, { json, static } from 'express';
const sqlite3 = require('sqlite3').verbose();
import { join } from 'path';

const app = express();
const PORT = 3000;
const db = new sqlite3.Database('./database.sqlite', (err) => {
    if (err) {
        console.error('Ошибка подключения к БД:', err.message);
    } else {
        console.log('Подключено к SQLite базе данных.');
        db.run(`CREATE TABLE IF NOT EXISTS items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            description TEXT
        )`, (err) => {
            if (err) {
                console.error('Ошибка создания таблицы:', err.message);
            } else {
                console.log('Таблица "items" готова.');
                db.get(`SELECT COUNT(*) AS count FROM items`, (err, row) => {
                    if (err) {
                        console.error('Ошибка проверки пустоты таблицы:', err.message);
                    } else if (row.count === 0) {
                        const seedData = [
                            ['Пример записи 1', 'Это описание первой записи'],
                            ['Пример записи 2', 'Вторая запись для демонстрации']
                        ];
                        const stmt = db.prepare(`INSERT INTO items (name, description) VALUES (?, ?)`);
                        seedData.forEach(item => {
                            stmt.run(item[0], item[1], (err) => {
                                if (err) console.error('Ошибка вставки начальных данных:', err.message);
                            });
                        });
                        stmt.finalize();
                        console.log('Добавлены начальные записи в таблицу.');
                    }
                });
            }
        });
    }
});
app.use(json());
app.use(static(join(__dirname, 'public')));
app.get('/api/items', (req, res) => {
    db.all(`SELECT * FROM items ORDER BY id DESC`, [], (err, rows) => {
        if (err) {
            res.status(500).json({ error: err.message });
            return;
        }
        res.json(rows);
    });
});
app.post('/api/items', (req, res) => {
    const { name, description } = req.body;
    if (!name) {
        res.status(400).json({ error: 'Поле name обязательно' });
        return;
    }
    db.run(`INSERT INTO items (name, description) VALUES (?, ?)`, [name, description], function (err) {
        if (err) {
            res.status(500).json({ error: err.message });
            return;
        }
        res.status(201).json({ id: this.lastID, name, description });
    });
});
app.listen(PORT, () => {
    console.log(`Сервер запущен на http://localhost:${PORT}`);
});



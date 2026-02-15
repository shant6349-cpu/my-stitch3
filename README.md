<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch Messenger</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    
    <style>
        /* Обнуляем всё, чтобы заняло весь экран */
        html, body { 
            margin: 0; 
            padding: 0; 
            width: 100% !important; 
            height: 100% !important; 
            overflow: hidden; 
            position: fixed; /* Фиксируем, чтобы не дергалось */
            font-family: -apple-system, sans-serif;
        }

        /* Экран входа */
        #auth-screen { 
            position: fixed; 
            inset: 0; 
            background: white; 
            z-index: 1000; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
        }

        /* Главное окно на ВЕСЬ экран */
        #chat-window { 
            display: none; 
            flex-direction: column; 
            width: 100vw !important; 
            height: 100vh !important; 
            background: #c7d2d9 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); 
            background-size: cover; 
        }

        header { 
            background: #3390ec; 
            color: white; 
            padding: 15px; 
            width: 100%;
            display: flex; 
            justify-content: space-between; 
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        #messages { 
            flex: 1; 
            width: 100% !important; 
            padding: 10px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 10px; 
        }

        /* Сообщения теперь реально широкие */
        .msg { 
            max-width: 88% !important; 
            padding: 10px 14px; 
            border-radius: 18px; 
            font-size: 16px; 
            word-wrap: break-word; 
            position: relative;
            box-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        .msg.sent { align-self: flex-end; background: #effdde; border-bottom-right-radius: 2px; }
        .msg.received { align-self: flex-start; background: #ffffff; border-bottom-left-radius: 2px; }

        .msg-info { font-size: 11px; color: #888; margin-bottom: 3px; display: block; }

        /* Поле ввода внизу */
        .input-area { 
            background: #fff; 
            padding: 10px; 
            display: flex; 
            width: 100%;
            gap: 10px; 
            border-top: 1px solid #ddd;
        }
        .input-area input { 
            flex: 1; 
            padding: 12px; 
            border: none; 
            background: #f1f1f1; 
            border-radius: 25px; 
            outline: none; 
            font-size: 16px;
        }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div style="width: 80%; text-align: center;">
            <h1 style="color: #3390ec; margin-bottom: 20px;">Stitch</h1>
            <input type="text" id="username" id="username" placeholder="Твоё имя..." 
                   style="width: 100%; padding: 15px; border: 1px solid #ddd; border-radius: 10px; margin-bottom: 10px;">
            <button onclick="login()" 
                    style="width: 100%; padding: 15px; background: #3390ec; color: white; border: none; border-radius: 10px; font-weight: bold;">Войти</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <b>Stitch Messenger</b>
            <i class="fa-solid fa-sync" onclick="location.reload()"></i>
        </header>
        <div id="messages"></div>
        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <button onclick="send()" style="background: none; border: none; color: #3390ec; font-size: 24px;"><i class="fa-solid fa-paper-plane"></i></button>
        </div>
    </div>

<script>
    const firebaseConfig = {
        apiKey: "AIzaSyCfAgY00uE-PoAK0esojcsQMeTdiA3PJXE",
        authDomain: "mesenjershant.firebaseapp.com",
        databaseURL: "https://mesenjershant-default-rtdb.firebaseio.com",
        projectId: "mesenjershant",
        storageBucket: "mesenjershant.firebasestorage.app",
        messagingSenderId: "509135555574",
        appId: "1:509135555574:web:6845e17523e44bae2215d7"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    let myName = "";

    function login() {
        myName = document.getElementById('username').value.trim();
        if (myName.length < 2) return;
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('chat-window').style.display = 'flex';
        loadMessages();
    }

    function send() {
        const input = document.getElementById('msg-input');
        if (input.value.trim() === "") return;
        db.ref('messages').push({
            user: myName,
            text: input.value,
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})
        });
        input.value = "";
    }

    function loadMessages() {
        db.ref('messages').on('value', (snapshot) => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            const data = snapshot.val();
            if (data) {
                Object.values(data).forEach(m => {
                    const div = document.createElement('div');
                    div.className = `msg ${m.user === myName ? 'sent' : 'received'}`;
                    div.innerHTML = `<span class="msg-info"><b>${m.user}</b></span>${m.text}`;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch Premium</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root {
            --primary: #0088cc;
            --bg-gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            --glass: rgba(255, 255, 255, 0.9);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        
        body { 
            font-family: 'Segoe UI', Roboto, Helvetica, sans-serif; 
            background: var(--bg-gradient);
            height: 100dvh;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        /* Экран входа */
        #auth-screen {
            position: fixed;
            inset: 0;
            background: var(--bg-gradient);
            z-index: 100;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .login-card {
            background: var(--glass);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            text-align: center;
            width: 90%;
            max-width: 350px;
            backdrop-filter: blur(10px);
        }

        input {
            width: 100%;
            padding: 15px;
            margin: 15px 0;
            border: 1px solid #ddd;
            border-radius: 12px;
            font-size: 16px;
            outline: none;
        }

        button {
            width: 100%;
            padding: 15px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: 0.3s;
        }

        button:active { transform: scale(0.98); }

        /* Окно чата */
        #chat-window {
            display: none;
            flex-direction: column;
            width: 100%;
            max-width: 600px;
            height: 95dvh;
            background: var(--glass);
            border-radius: 20px;
            overflow: hidden;
            box-shadow: 0 20px 50px rgba(0,0,0,0.3);
            backdrop-filter: blur(15px);
            margin: 10px;
        }

        header {
            background: var(--primary);
            color: white;
            padding: 15px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        #messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
            background: rgba(255,255,255,0.3);
        }

        .msg {
            max-width: 85%;
            padding: 12px 16px;
            border-radius: 18px;
            font-size: 15px;
            position: relative;
            animation: fadeIn 0.3s ease;
            line-height: 1.4;
        }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .msg.sent { align-self: flex-end; background: #d1eaff; color: #111; border-bottom-right-radius: 4px; }
        .msg.received { align-self: flex-start; background: #fff; color: #111; border-bottom-left-radius: 4px; }

        .msg-user { font-size: 11px; font-weight: bold; margin-bottom: 4px; display: block; color: var(--primary); }
        .msg-time { font-size: 10px; opacity: 0.6; display: block; text-align: right; margin-top: 4px; }

        /* Область ввода */
        .input-area {
            padding: 15px;
            background: #fff;
            display: flex;
            gap: 10px;
            align-items: center;
            border-top: 1px solid rgba(0,0,0,0.05);
        }

        #msg-input { margin: 0; background: #f0f2f5; border: none; }

        .send-btn {
            width: 45px;
            height: 45px;
            padding: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
        }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="login-card">
            <i class="fa-solid fa-comments" style="font-size: 50px; color: var(--primary); margin-bottom: 10px;"></i>
            <h2 style="color: #333;">Stitch Connect</h2>
            <input type="text" id="username" placeholder="Придумай имя...">
            <button onclick="login()">Войти в чат</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <div style="display: flex; align-items: center; gap: 10px;">
                <div id="avatar" style="width: 35px; height: 35px; background: #fff; border-radius: 50%; color: var(--primary); display: flex; align-items: center; justify-content: center; font-weight: bold;">?</div>
                <b id="user-display">Чат</b>
            </div>
            <div style="display: flex; gap: 15px; font-size: 18px;">
                <i class="fa-solid fa-trash" onclick="deleteChat()" style="cursor: pointer;"></i>
                <i class="fa-solid fa-right-from-bracket" onclick="location.reload()" style="cursor: pointer;"></i>
            </div>
        </header>

        <div id="messages"></div>

        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Написать сообщение..." onkeypress="if(event.key==='Enter') send()">
            <button class="send-btn" onclick="send()">
                <i class="fa-solid fa-paper-plane"></i>
            </button>
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
        document.getElementById('user-display').innerText = myName;
        document.getElementById('avatar').innerText = myName[0].toUpperCase();
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

    function deleteChat() {
        if(confirm("Очистить историю для всех?")) db.ref('messages').remove();
    }

    function loadMessages() {
        db.ref('messages').on('value', (snapshot) => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            const data = snapshot.val();
            if (data) {
                Object.values(data).forEach(m => {
                    const isMe = m.user === myName;
                    const div = document.createElement('div');
                    div.className = `msg ${isMe ? 'sent' : 'received'}`;
                    div.innerHTML = `
                        <span class="msg-user">${isMe ? 'Вы' : m.user}</span>
                        ${m.text}
                        <span class="msg-time">${m.time}</span>
                    `;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }
</script>
</body>
</html>

<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch Connect Premium</title>
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
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; 
            background: var(--bg-gradient);
            height: 100dvh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
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

        .login-btn {
            width: 100%;
            padding: 15px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
        }

        /* Окно чата */
        #chat-window {
            display: none;
            flex-direction: column;
            width: 100%;
            max-width: 600px;
            height: 98dvh;
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

        .header-btns { display: flex; gap: 15px; font-size: 18px; }

        #messages {
            flex: 1;
            padding: 15px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 12px;
            background: rgba(255,255,255,0.2);
        }

        .msg {
            max-width: 85%;
            padding: 10px 14px;
            border-radius: 15px;
            font-size: 16px;
            position: relative;
            animation: fadeIn 0.3s ease;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
        }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }

        .msg.sent { align-self: flex-end; background: #d1eaff; border-bottom-right-radius: 2px; }
        .msg.received { align-self: flex-start; background: #fff; border-bottom-left-radius: 2px; }

        .msg-user { font-size: 11px; font-weight: bold; margin-bottom: 4px; display: flex; justify-content: space-between; align-items: center; color: var(--primary); }
        .msg-time { font-size: 10px; opacity: 0.5; margin-top: 4px; text-align: right; }

        .edit-btn { margin-left: 8px; cursor: pointer; color: #666; font-size: 12px; }
        .edit-btn:hover { color: var(--primary); }

        /* Область ввода */
        .input-area {
            padding: 12px 15px;
            background: #fff;
            display: flex;
            gap: 10px;
            align-items: center;
            border-top: 1px solid rgba(0,0,0,0.05);
            padding-bottom: env(safe-area-inset-bottom, 12px);
        }

        #msg-input { margin: 0; background: #f0f2f5; border: none; border-radius: 25px; }

        .send-btn {
            width: 45px;
            height: 45px;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="login-card">
            <h2 style="color: #333; margin-bottom: 10px;">Stitch Connect</h2>
            <p style="color: gray; font-size: 14px;">Введите ник, чтобы войти</p>
            <input type="text" id="username" placeholder="Ваш ник...">
            <button class="login-btn" onclick="login()">Войти</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <b id="user-display">Чат</b>
            <div class="header-btns">
                <i class="fa-solid fa-trash" onclick="deleteChat()" style="cursor: pointer;" title="Очистить всё"></i>
                <i class="fa-solid fa-power-off" onclick="location.reload()" style="cursor: pointer;"></i>
            </div>
        </header>

        <div id="messages"></div>

        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
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
    let editId = null; 

    function login() {
        myName = document.getElementById('username').value.trim();
        if (myName.length < 2) return alert("Ник слишком короткий");
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('chat-window').style.display = 'flex';
        document.getElementById('user-display').innerText = "Я: " + myName;
        loadMessages();
    }

    function send() {
        const input = document.getElementById('msg-input');
        const text = input.value.trim();
        if (text === "") return;

        if (editId) {
            db.ref('messages/' + editId).update({
                text: text + " (изм.)"
            });
            editId = null;
        } else {
            db.ref('messages').push({
                user: myName,
                text: text,
                time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})
            });
        }
        input.value = "";
    }

    window.startEdit = function(id, text) {
        const input = document.getElementById('msg-input');
        input.value = text.replace(" (изм.)", "");
        input.focus();
        editId = id;
    }

    function deleteChat() {
        if(confirm("Удалить все сообщения?")) db.ref('messages').remove();
    }

    function loadMessages() {
        db.ref('messages').on('value', (snapshot) => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            const data = snapshot.val();
            if (data) {
                Object.entries(data).forEach(([id, m]) => {
                    const isMe = m.user === myName;
                    const div = document.createElement('div');
                    div.className = `msg ${isMe ? 'sent' : 'received'}`;
                    
                    const editIcon = isMe ? `<i class="fa-solid fa-pen edit-btn" onclick="startEdit('${id}', '${m.text}')"></i>` : '';
                    
                    div.innerHTML = `
                        <span class="msg-user">
                            ${isMe ? 'Вы' : m.user} 
                            ${editIcon}
                        </span>
                        <div class="msg-text">${m.text}</div>
                        <div class="msg-time">${m.time}</div>
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

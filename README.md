<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch Online</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        
        html, body { 
            width: 100%; 
            height: 100%; 
            overflow: hidden; 
            background: #e7ebf0; 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; 
        }
        
        #auth-screen { 
            position: fixed; 
            inset: 0; 
            background: white; 
            z-index: 1000; 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            justify-content: center; 
        }
        .auth-box { width: 300px; text-align: center; }
        input#username { width: 100%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; outline: none; font-size: 16px; }
        .login-btn { width: 100%; padding: 12px; background: #3390ec; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }

        header { 
            background: #3390ec; 
            color: white; 
            padding: 15px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            width: 100%; 
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .header-btns { display: flex; gap: 18px; align-items: center; font-size: 20px; }
        
        #chat-window { 
            display: none; 
            flex-direction: column; 
            width: 100vw; 
            height: 100dvh; /* Динамическая высота для iPhone */
            background: url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); 
            background-size: cover; 
        }

        #messages { 
            flex: 1; 
            padding: 12px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 8px; 
            width: 100%; 
        }
        
        .msg { 
            max-width: 90%; 
            min-width: 80px;
            padding: 8px 12px; 
            border-radius: 15px; 
            font-size: 16px; 
            box-shadow: 0 1px 2px rgba(0,0,0,0.1); 
            word-wrap: break-word; 
            position: relative;
        }

        .msg.sent { align-self: flex-end; background: #effdde; border-bottom-right-radius: 2px; }
        .msg.received { align-self: flex-start; background: #fff; border-bottom-left-radius: 2px; }
        
        .msg-info { font-size: 11px; color: #888; margin-bottom: 2px; display: block; }

        .input-area { 
            background: #fff; 
            padding: 10px 15px; 
            padding-bottom: env(safe-area-inset-bottom, 10px); /* Отступ для новых iPhone */
            display: flex; 
            gap: 12px; 
            align-items: center; 
            border-top: 1px solid #ddd; 
            width: 100%;
        }
        .input-area input { 
            flex: 1; 
            border: none; 
            background: #f1f1f1; 
            border-radius: 25px; 
            padding: 12px 18px; 
            font-size: 16px;
            outline: none;
        }
        .send-icon { color: #3390ec; font-size: 26px; cursor: pointer; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-box">
            <h1 style="color: #3390ec; margin-bottom: 5px;">Stitch Online</h1>
            <p style="color: gray; font-size: 14px; margin-bottom: 20px;">Версия 2.0 (Wide)</p>
            <input type="text" id="username" placeholder="Твой ник">
            <button class="login-btn" onclick="login()">Начать общение</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <b id="user-display">Чат</b>
            <div class="header-btns">
                <i class="fa-solid fa-trash" onclick="deleteChat()" style="cursor: pointer;"></i>
                <i class="fa-solid fa-power-off" onclick="location.reload()" style="cursor: pointer;"></i>
            </div>
        </header>
        <div id="messages"></div>
        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane send-icon" onclick="send()"></i>
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
        if (myName.length < 2) return alert("Введите ник!");
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('chat-window').style.display = 'flex';
        document.getElementById('user-display').innerText = "Я: " + myName;
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
        if (confirm("Удалить всю историю сообщений?")) {
            db.ref('messages').remove();
        }
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
                    div.innerHTML = `<div class="msg-info"><b>${m.user}</b> • ${m.time}</div>${m.text}`;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }
</script>
</body>
</html>

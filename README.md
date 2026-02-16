<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stitch Online Messenger</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, sans-serif; background: #e7ebf0; height: 100vh; display: flex; flex-direction: column; }
        
        #auth-screen { position: fixed; inset: 0; background: white; z-index: 1000; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .auth-box { width: 300px; text-align: center; }
        input { width: 100%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; outline: none; }
        button { width: 100%; padding: 12px; background: #3390ec; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }

        header { background: #3390ec; color: white; padding: 15px; display: flex; justify-content: space-between; align-items: center; }
        .header-btns { display: flex; gap: 20px; align-items: center; }
        
        #chat-window { flex: 1; display: none; flex-direction: column; background: url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); background-size: cover; }
        #messages { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
        
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 12px; font-size: 15px; box-shadow: 0 1px 2px rgba(0,0,0,0.1); word-wrap: break-word; }
        .msg.sent { align-self: flex-end; background: #effdde; border-bottom-right-radius: 2px; }
        .msg.received { align-self: flex-start; background: #fff; border-bottom-left-radius: 2px; }
        .msg-info { font-size: 10px; color: #888; margin-bottom: 2px; }

        .input-area { background: #fff; padding: 10px; display: flex; gap: 10px; align-items: center; border-top: 1px solid #ddd; }
        .input-area input { flex: 1; margin: 0; border: none; background: #f1f1f1; border-radius: 20px; padding-left: 15px; font-size: 16px; outline: none; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-box">
            <h1 style="font-family: cursive; color: #3390ec; margin-bottom: 10px;">Stitch Online</h1>
            <p style="color: gray; font-size: 14px;">Никаких ботов, только онлайн</p>
            <input type="text" id="username" placeholder="Твой ник">
            <button onclick="login()">Начать общение</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <b id="user-display">Чат</b>
            <div class="header-btns">
                <i class="fa-solid fa-trash" onclick="deleteChat()" style="cursor: pointer;" title="Очистить весь чат"></i>
                <i class="fa-solid fa-power-off" onclick="location.reload()" style="cursor: pointer;" title="Выйти"></i>
            </div>
        </header>
        <div id="messages"></div>
        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Написать..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" style="color:#3390ec; font-size: 24px; cursor: pointer;" onclick="send()"></i>
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
        if (myName.length < 2) return alert("Придумай ник подлиннее!");
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

    // НОВАЯ ФУНКЦИЯ: Удаление всех сообщений из базы
    function deleteChat() {
        if (confirm("Ты уверен, что хочешь удалить ВСЕ сообщения в этом чате? Это нельзя отменить.")) {
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

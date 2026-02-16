<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch TG</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --tg-blue: #0088cc; --tg-bg: #ffffff; --tg-gray: #f4f4f5; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        
        body { background: #e7ebf0; height: 100dvh; display: flex; justify-content: center; }

        /* Экраны */
        .screen { position: fixed; inset: 0; display: none; background: white; z-index: 10; }
        .active { display: flex; flex-direction: column; }

        /* Вход */
        #auth-screen { background: var(--tg-blue); justify-content: center; align-items: center; }
        .login-box { background: white; padding: 30px; border-radius: 20px; width: 90%; max-width: 350px; text-align: center; }
        
        /* Список чатов (Главная как в ТГ) */
        header { background: var(--tg-blue); color: white; padding: 15px; display: flex; align-items: center; gap: 15px; }
        .search-container { padding: 10px; background: white; border-bottom: 1px solid #eee; }
        .search-bar { background: var(--tg-gray); border-radius: 10px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .search-bar input { border: none; background: none; outline: none; width: 100%; font-size: 16px; }

        #user-list { flex: 1; overflow-y: auto; }
        .user-item { 
            display: flex; align-items: center; gap: 15px; padding: 12px 15px; 
            cursor: pointer; transition: 0.2s; border-bottom: 1px solid #f9f9f9;
        }
        .user-item:hover { background: var(--tg-gray); }
        .avatar { width: 50px; height: 50px; background: #accbee; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 20px; }
        .user-info { flex: 1; }
        .user-info b { font-size: 16px; color: #000; }
        .user-info p { font-size: 13px; color: #888; }

        /* Окно чата */
        #chat-window { z-index: 20; }
        #messages { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 8px; background: url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); background-size: cover; }
        
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 15px; font-size: 15px; line-height: 1.4; }
        .msg.sent { align-self: flex-end; background: #effdde; border-bottom-right-radius: 2px; box-shadow: 0 1px 1px rgba(0,0,0,0.1); }
        .msg.received { align-self: flex-start; background: white; border-bottom-left-radius: 2px; box-shadow: 0 1px 1px rgba(0,0,0,0.1); }

        .input-area { padding: 10px; display: flex; gap: 10px; align-items: center; background: white; }
        .input-area input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; font-size: 16px; }
    </style>
</head>
<body>

    <div id="auth-screen" class="screen active">
        <div class="login-box">
            <h2 style="color:var(--tg-blue); margin-bottom:15px;">Stitch Messenger</h2>
            <input type="text" id="username" placeholder="Ваше имя (Юзернейм)" style="width:100%; padding:12px; margin-bottom:10px; border-radius:10px; border:1px solid #ddd;">
            <button onclick="login()" style="width:100%; padding:12px; background:var(--tg-blue); color:white; border:none; border-radius:10px; font-weight:bold;">Далее</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header>
            <i class="fa-solid fa-bars"></i>
            <b style="font-size: 20px;">StitchGram</b>
        </header>
        <div class="search-container">
            <div class="search-bar">
                <i class="fa-solid fa-magnifying-glass" style="color:#888;"></i>
                <input type="text" id="search-input" placeholder="Поиск людей..." oninput="filterUsers()">
            </div>
        </div>
        <div id="user-list">
            </div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left" onclick="closeChat()"></i>
            <div class="avatar" id="chat-avatar" style="width:35px; height:35px; font-size:14px;">?</div>
            <b id="chat-with-name">Имя</b>
        </header>
        <div id="messages"></div>
        <div class="input-area">
            <i class="fa-solid fa-paperclip" style="color:#888; font-size:20px;"></i>
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()" style="color:var(--tg-blue); font-size:22px; cursor:pointer;"></i>
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
    let chatPartner = "";
    let chatID = "";
    let allUsers = {};

    function login() {
        myName = document.getElementById('username').value.trim();
        if (myName.length < 2) return;

        // Регистрируем юзера в базе
        db.ref('users/' + myName).set({ name: myName, online: true });
        db.ref('users/' + myName).onDisconnect().remove();

        document.getElementById('auth-screen').classList.remove('active');
        document.getElementById('menu-screen').classList.add('active');

        listenUsers();
    }

    function listenUsers() {
        db.ref('users').on('value', (snap) => {
            allUsers = snap.val() || {};
            renderUserList(allUsers);
        });
    }

    function renderUserList(users) {
        const list = document.getElementById('user-list');
        list.innerHTML = '';
        Object.keys(users).forEach(user => {
            if (user !== myName) {
                const div = document.createElement('div');
                div.className = 'user-item';
                div.onclick = () => openChat(user);
                div.innerHTML = `
                    <div class="avatar">${user[0].toUpperCase()}</div>
                    <div class="user-info">
                        <b>${user}</b>
                        <p>был(а) недавно</p>
                    </div>
                `;
                list.appendChild(div);
            }
        });
    }

    function filterUsers() {
        const query = document.getElementById('search-input').value.toLowerCase();
        const filtered = {};
        Object.keys(allUsers).forEach(user => {
            if (user.toLowerCase().includes(query)) {
                filtered[user] = allUsers[user];
            }
        });
        renderUserList(filtered);
    }

    function openChat(partner) {
        chatPartner = partner;
        chatID = [myName, chatPartner].sort().join("_");
        
        document.getElementById('chat-with-name').innerText = chatPartner;
        document.getElementById('chat-avatar').innerText = chatPartner[0].toUpperCase();
        document.getElementById('chat-window').classList.add('active');
        
        loadMessages();
    }

    function closeChat() {
        document.getElementById('chat-window').classList.remove('active');
        db.ref('privates/' + chatID).off();
    }

    function send() {
        const input = document.getElementById('msg-input');
        if (input.value.trim() === "") return;
        db.ref('privates/' + chatID).push({
            sender: myName,
            text: input.value,
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})
        });
        input.value = "";
    }

    function loadMessages() {
        db.ref('privates/' + chatID).on('value', (snap) => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            const data = snap.val();
            if (data) {
                Object.values(data).forEach(m => {
                    const isMe = m.sender === myName;
                    const div = document.createElement('div');
                    div.className = `msg ${isMe ? 'sent' : 'received'}`;
                    div.innerHTML = `${m.text} <div style="font-size:10px; opacity:0.5; text-align:right;">${m.time}</div>`;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }
</script>
</body>
</html>

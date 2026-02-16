<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch-Gram Private Control</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --tg-blue: #248bcf; --tg-bg: #e7ebf0; --white: #ffffff; --gray: #8e8e93; --red: #ff4d4d; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, sans-serif; }
        body { background: var(--tg-bg); height: 100dvh; display: flex; justify-content: center; overflow: hidden; }

        .screen { position: fixed; inset: 0; display: none; background: white; z-index: 10; flex-direction: column; }
        .active { display: flex; }

        header { background: var(--tg-blue); color: white; padding: 12px 15px; display: flex; align-items: center; gap: 15px; }
        
        /* Поиск */
        .search-area { padding: 10px; border-bottom: 1px solid #eee; }
        .search-box { background: #f2f2f7; border-radius: 10px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .search-box input { border: none; background: none; outline: none; width: 100%; font-size: 16px; }

        /* Список */
        .section-title { padding: 10px 15px; font-size: 13px; color: var(--tg-blue); font-weight: bold; background: #f9f9f9; }
        .user-item { display: flex; align-items: center; gap: 12px; padding: 12px 15px; cursor: pointer; border-bottom: 1px solid #f2f2f2; position: relative; }
        .avatar { width: 45px; height: 45px; background: #accbee; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; }
        
        .action-btns { margin-left: auto; display: flex; gap: 15px; }
        .add-btn { color: var(--tg-blue); font-size: 20px; }
        .del-btn { color: var(--red); font-size: 18px; padding: 5px; opacity: 0.6; }
        .del-btn:hover { opacity: 1; }

        /* Чат */
        #messages { flex: 1; padding: 15px; overflow-y: auto; background: #96afc3 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); display: flex; flex-direction: column; gap: 8px; }
        .msg { max-width: 75%; padding: 8px 12px; border-radius: 15px; font-size: 15px; }
        .msg.sent { align-self: flex-end; background: #effdde; }
        .msg.received { align-self: flex-start; background: white; }

        .bottom-bar { background: white; padding: 8px 12px; display: flex; align-items: center; gap: 12px; border-top: 1px solid #ddd; }
        #msg-input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; background: #f4f4f5; }
    </style>
</head>
<body>

    <div id="auth-screen" class="screen active" style="justify-content:center; align-items:center; background:var(--tg-blue);">
        <div style="background:white; padding:30px; border-radius:20px; text-align:center; width:90%; max-width:320px;">
            <h2>Stitch-Gram</h2>
            <input type="text" id="username" placeholder="Твой ник" style="width:100%; padding:10px; margin:15px 0; border-radius:10px; border:1px solid #ddd;">
            <button onclick="login()" style="width:100%; padding:10px; background:var(--tg-blue); color:white; border:none; border-radius:10px; font-weight:bold;">Войти</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header><b>Контакты</b></header>
        <div class="search-area">
            <div class="search-box">
                <i class="fa-solid fa-magnifying-glass"></i>
                <input type="text" id="search-in" placeholder="Найти по нику..." oninput="searchPeople()">
            </div>
        </div>
        <div id="search-results"></div>
        <div class="section-title">МОИ ДРУЗЬЯ</div>
        <div id="friends-list"></div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left" onclick="closeChat()" style="cursor:pointer"></i>
            <b id="h-name">Чат</b>
        </header>
        <div id="messages"></div>
        <div class="bottom-bar">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()" style="color:var(--tg-blue); font-size:22px; cursor:pointer"></i>
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
    let myName = "", chatPartner = "", chatID = "";
    let friends = JSON.parse(localStorage.getItem('myFriends') || '[]');

    async function login() {
        myName = document.getElementById('username').value.trim();
        if(!myName) return;
        
        const snap = await db.ref('users/' + myName).get();
        if(snap.exists()) {
             // Просто заходим, если ник наш, или выдаем предупреждение
        }

        await db.ref('users/' + myName).set({online: true});
        db.ref('users/' + myName).onDisconnect().remove();
        document.getElementById('auth-screen').classList.remove('active');
        document.getElementById('menu-screen').classList.add('active');
        renderFriends();
    }

    function searchPeople() {
        const q = document.getElementById('search-in').value.trim();
        const resultsArea = document.getElementById('search-results');
        resultsArea.innerHTML = '';
        if(!q) return;

        db.ref('users').orderByKey().startAt(q).endAt(q + "\uf8ff").once('value', snap => {
            const users = snap.val();
            if(users) {
                Object.keys(users).forEach(u => {
                    if(u !== myName && !friends.includes(u)) {
                        const div = document.createElement('div');
                        div.className = 'user-item';
                        div.innerHTML = `
                            <div class="avatar">${u[0].toUpperCase()}</div>
                            <b>${u}</b> 
                            <div class="action-btns">
                                <i class="fa-solid fa-user-plus add-btn" onclick="addFriend('${u}')"></i>
                            </div>`;
                        resultsArea.appendChild(div);
                    }
                });
            }
        });
    }

    function addFriend(name) {
        if(!friends.includes(name)) {
            friends.push(name);
            saveFriends();
            document.getElementById('search-in').value = '';
            document.getElementById('search-results').innerHTML = '';
            renderFriends();
        }
    }

    function removeFriend(name, event) {
        event.stopPropagation(); // Чтобы не открылся чат при нажатии на корзину
        if(confirm(`Удалить ${name} из друзей?`)) {
            friends = friends.filter(f => f !== name);
            saveFriends();
            renderFriends();
        }
    }

    function saveFriends() {
        localStorage.setItem('myFriends', JSON.stringify(friends));
    }

    function renderFriends() {
        const list = document.getElementById('friends-list');
        list.innerHTML = '';
        friends.forEach(f => {
            const div = document.createElement('div');
            div.className = 'user-item';
            div.onclick = () => openChat(f);
            div.innerHTML = `
                <div class="avatar" style="background:var(--tg-blue)">${f[0].toUpperCase()}</div>
                <b>${f}</b>
                <div class="action-btns">
                    <i class="fa-solid fa-trash-can del-btn" onclick="removeFriend('${f}', event)"></i>
                </div>`;
            list.appendChild(div);
        });
    }

    function openChat(partner) {
        chatPartner = partner;
        chatID = [myName, chatPartner].sort().join("_");
        document.getElementById('h-name').innerText = chatPartner;
        document.getElementById('chat-window').classList.add('active');
        loadMessages();
    }

    function send() {
        const input = document.getElementById('msg-input');
        if(!input.value.trim()) return;
        db.ref('privates/' + chatID).push({
            s: myName,
            t: input.value,
            time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})
        });
        input.value = "";
    }

    function loadMessages() {
        db.ref('privates/' + chatID).on('value', snap => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            const val = snap.val();
            if(val) {
                Object.values(val).forEach(m => {
                    const div = document.createElement('div');
                    div.className = `msg ${m.s === myName ? 'sent' : 'received'}`;
                    div.innerHTML = `${m.t} <div style="font-size:9px; opacity:0.5; text-align:right;">${m.time}</div>`;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }

    function closeChat() {
        document.getElementById('chat-window').classList.remove('active');
        db.ref('privates/' + chatID).off();
    }
</script>
</body>
</html>

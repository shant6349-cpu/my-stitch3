<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch-Gram Mobile Fixed</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --tg-blue: #248bcf; --tg-bg: #e7ebf0; --white: #ffffff; --gray: #8e8e93; --red: #ff4d4d; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, sans-serif; }
        
        body { background: var(--tg-bg); height: 100dvh; display: flex; justify-content: center; overflow: hidden; }

        .screen { position: fixed; inset: 0; display: none; background: white; z-index: 10; flex-direction: column; }
        .active { display: flex; }

        header { background: var(--tg-blue); color: white; padding: 12px 15px; display: flex; align-items: center; gap: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); z-index: 100; }
        
        /* AUTH */
        #auth-screen { background: var(--tg-blue); justify-content: center; align-items: center; }
        .auth-card { background: white; padding: 30px; border-radius: 20px; width: 90%; max-width: 320px; text-align: center; }
        .auth-card input { width: 100%; padding: 12px; margin: 15px 0; border: 1px solid #ddd; border-radius: 10px; outline: none; }
        .auth-card button { width: 100%; padding: 12px; background: var(--tg-blue); color: white; border: none; border-radius: 10px; font-weight: bold; }

        /* SEARCH & LIST */
        .search-area { padding: 10px; border-bottom: 1px solid #eee; }
        .search-box { background: #f2f2f7; border-radius: 10px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .search-box input { border: none; background: none; outline: none; width: 100%; font-size: 16px; }
        .user-item { display: flex; align-items: center; gap: 12px; padding: 12px 15px; border-bottom: 1px solid #f2f2f2; }
        .avatar { width: 45px; height: 45px; background: linear-gradient(135deg, #52a0ff, #007aff); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; }
        .item-actions { margin-left: auto; display: flex; gap: 15px; align-items: center; }

        /* CHAT */
        #messages { flex: 1; padding: 15px; overflow-y: auto; background: #96afc3 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); display: flex; flex-direction: column; gap: 8px; }
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 15px; font-size: 15px; box-shadow: 0 1px 2px rgba(0,0,0,0.1); }
        .msg.sent { align-self: flex-end; background: #effdde; }
        .msg.received { align-self: flex-start; background: white; }

        /* INPUT BAR */
        .bottom-bar { background: white; padding: 8px 12px; display: flex; align-items: center; gap: 10px; border-top: 1px solid #ddd; }
        #msg-input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; background: #f4f4f5; font-size: 16px; }

        /* EMOJI PANEL - FIXED FOR MOBILE */
        #emoji-picker { 
            display: none; 
            background: #f4f4f5; 
            border-top: 1px solid #ddd; 
            padding: 10px; 
            grid-template-columns: repeat(auto-fill, minmax(45px, 1fr)); /* Авто-подбор колонок */
            gap: 5px; 
            max-height: 200px; 
            overflow-y: auto; 
            -webkit-overflow-scrolling: touch;
        }
        .emoji-item { 
            font-size: 30px; /* Крупно для пальца */
            text-align: center; 
            padding: 5px; 
            cursor: pointer;
            user-select: none;
        }
        .emoji-item:active { background: #e2e2e2; border-radius: 10px; }

        @keyframes blink { 0% { opacity: .2; } 20% { opacity: 1; } 100% { opacity: .2; } }
        .typing span { animation: blink 1.4s infinite both; }
    </style>
</head>
<body>

    <audio id="notif-sound" src="https://www.soundjay.com/buttons/sounds/button-21.mp3"></audio>

    <div id="auth-screen" class="screen active">
        <div class="auth-card">
            <h2 style="color:var(--tg-blue)">Stitch-Gram</h2>
            <input type="text" id="username" placeholder="Ваш ник">
            <button onclick="login()">Войти</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header><b>Контакты</b></header>
        <div class="search-area">
            <div class="search-box">
                <i class="fa-solid fa-magnifying-glass" style="color:var(--gray)"></i>
                <input type="text" id="search-in" placeholder="Найти по нику..." oninput="searchUsers()">
            </div>
        </div>
        <div id="search-results"></div>
        <div style="padding:10px 15px; font-size:12px; color:var(--tg-blue); font-weight:bold; background:#f9f9f9;">МОИ ДРУЗЬЯ</div>
        <div id="friends-list" style="flex:1; overflow-y:auto;"></div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left" onclick="closeChat()"></i>
            <div style="flex:1">
                <div id="h-name" style="font-weight:bold;">Чат</div>
                <div id="h-status" style="font-size:11px; opacity:0.8;"></div>
            </div>
        </header>
        <div id="messages"></div>
        <div id="emoji-picker"></div>
        <div class="bottom-bar">
            <i class="fa-regular fa-face-smile" style="font-size:24px; color:var(--gray);" onclick="toggleEmoji()"></i>
            <input type="text" id="msg-input" placeholder="Сообщение..." oninput="isTyping()" onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()" style="color:var(--tg-blue); font-size:24px;"></i>
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
    let myName = "", chatPartner = "", chatID = "", typeTimer;
    let friends = JSON.parse(localStorage.getItem('myFriends') || '[]');

    async function login() {
        myName = document.getElementById('username').value.trim();
        if(!myName) return;
        await db.ref('users/' + myName).set({online: true});
        db.ref('users/' + myName).onDisconnect().remove();
        document.getElementById('auth-screen').classList.remove('active');
        document.getElementById('menu-screen').classList.add('active');
        renderFriends();
        initEmoji();
    }

    function searchUsers() {
        const q = document.getElementById('search-in').value.trim();
        const res = document.getElementById('search-results');
        res.innerHTML = '';
        if(!q) return;
        db.ref('users').orderByKey().startAt(q).endAt(q + "\uf8ff").once('value', snap => {
            const data = snap.val();
            if(data) {
                Object.keys(data).forEach(u => {
                    if(u !== myName && !friends.includes(u)) {
                        const div = document.createElement('div');
                        div.className = 'user-item';
                        div.innerHTML = `<div class="avatar">${u[0].toUpperCase()}</div><b>${u}</b>
                                         <div class="item-actions"><i class="fa-solid fa-plus" style="color:var(--tg-blue)" onclick="addFriend('${u}')"></i></div>`;
                        res.appendChild(div);
                    }
                });
            }
        });
    }

    function addFriend(u) {
        friends.push(u);
        localStorage.setItem('myFriends', JSON.stringify(friends));
        document.getElementById('search-in').value = '';
        document.getElementById('search-results').innerHTML = '';
        renderFriends();
    }

    function removeFriend(u, e) {
        e.stopPropagation();
        if(confirm('Удалить из друзей?')) {
            friends = friends.filter(f => f !== u);
            localStorage.setItem('myFriends', JSON.stringify(friends));
            renderFriends();
        }
    }

    function renderFriends() {
        const list = document.getElementById('friends-list');
        list.innerHTML = '';
        friends.forEach(f => {
            const div = document.createElement('div');
            div.className = 'user-item';
            div.onclick = () => openChat(f);
            div.innerHTML = `<div class="avatar" style="background:var(--tg-blue)">${f[0].toUpperCase()}</div><b>${f}</b>
                             <div class="item-actions"><i class="fa-solid fa-trash" style="color:var(--red); font-size:14px;" onclick="removeFriend('${f}', event)"></i></div>`;
            list.appendChild(div);
        });
    }

    function openChat(p) {
        chatPartner = p;
        chatID = [myName, chatPartner].sort().join("_");
        document.getElementById('h-name').innerText = chatPartner;
        document.getElementById('chat-window').classList.add('active');
        
        db.ref('typing/' + chatID + '/' + chatPartner).on('value', s => {
            document.getElementById('h-status').innerHTML = s.val() ? 'печатает...' : 'в сети';
        });
        loadMessages();
    }

    function send() {
        const input = document.getElementById('msg-input');
        if(!input.value.trim()) return;
        db.ref('privates/' + chatID).push({
            s: myName, t: input.value,
            time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})
        });
        input.value = "";
        document.getElementById('emoji-picker').style.display = 'none';
    }

    function loadMessages() {
        db.ref('privates/' + chatID).on('value', snap => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            if(snap.val()) {
                Object.values(snap.val()).forEach(m => {
                    const div = document.createElement('div');
                    div.className = `msg ${m.s === myName ? 'sent' : 'received'}`;
                    div.innerHTML = `${m.t} <div style="font-size:10px; opacity:0.4; text-align:right;">${m.time}</div>`;
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
                if(Object.values(snap.val()).pop().s !== myName) document.getElementById('notif-sound').play();
            }
        });
    }

    function isTyping() {
        db.ref('typing/' + chatID + '/' + myName).set(true);
        clearTimeout(typeTimer);
        typeTimer = setTimeout(() => db.ref('typing/' + chatID + '/' + myName).set(false), 2000);
    }

    function initEmoji() {
        const p = document.getElementById('emoji-picker');
        const list = ['😂','😍','😭','👍','🔥','❤️','😎','😊','🚀','👏','🤔','✨','✅','👋','🌚','🤡','💩','⚡️','🎉','🔔','💎','💰','🎮','🎁','🍭'];
        p.innerHTML = '';
        list.forEach(e => {
            const s = document.createElement('span');
            s.className = 'emoji-item';
            s.innerText = e;
            s.onclick = (ev) => {
                ev.preventDefault();
                document.getElementById('msg-input').value += e;
                document.getElementById('msg-input').focus();
            };
            p.appendChild(s);
        });
    }

    function toggleEmoji() {
        const p = document.getElementById('emoji-picker');
        p.style.display = p.style.display === 'grid' ? 'none' : 'grid';
    }

    function closeChat() {
        document.getElementById('chat-window').classList.remove('active');
        db.ref('privates/' + chatID).off();
    }
</script>
</body>
</html>

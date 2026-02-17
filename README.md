<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch-Gram Circle Fix</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --tg-blue: #248bcf; --tg-bg: #e7ebf0; --white: #ffffff; --gray: #8e8e93; --red: #ff4d4d; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, sans-serif; }
        body { background: var(--tg-bg); height: 100dvh; display: flex; justify-content: center; overflow: hidden; }

        .screen { position: fixed; inset: 0; display: none; background: white; z-index: 10; flex-direction: column; }
        .active { display: flex; }

        header { background: var(--tg-blue); color: white; padding: 12px 15px; display: flex; align-items: center; gap: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .back-btn { cursor: pointer; font-size: 20px; }

        /* Auth */
        #auth-screen { background: var(--tg-blue); justify-content: center; align-items: center; }
        .auth-card { background: white; padding: 30px; border-radius: 20px; width: 90%; max-width: 320px; text-align: center; }
        .auth-card input { width: 100%; padding: 12px; margin: 15px 0; border: 1px solid #ddd; border-radius: 10px; outline: none; }
        .auth-card button { width: 100%; padding: 12px; background: var(--tg-blue); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }

        /* List & Search */
        .search-area { padding: 10px; border-bottom: 1px solid #eee; }
        .search-box { background: #f2f2f7; border-radius: 10px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .search-box input { border: none; background: none; outline: none; width: 100%; font-size: 16px; }
        .user-item { display: flex; align-items: center; gap: 12px; padding: 12px 15px; border-bottom: 1px solid #f2f2f2; }
        .avatar { width: 45px; height: 45px; background: linear-gradient(135deg, #52a0ff, #007aff); border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; }
        .item-actions { margin-left: auto; display: flex; gap: 15px; }

        /* Chat */
        #messages { flex: 1; padding: 15px; overflow-y: auto; background: #96afc3 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); display: flex; flex-direction: column; gap: 8px; }
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 15px; font-size: 15px; background: white; box-shadow: 0 1px 2px rgba(0,0,0,0.1); position: relative; }
        .msg.sent { align-self: flex-end; background: #effdde; }
        
        /* КРУЖОЧЕК В ЧАТЕ */
        .circle-v { width: 200px; height: 200px; border-radius: 50%; object-fit: cover; border: 2px solid var(--tg-blue); background: black; display: block; }

        /* Bottom bar */
        .bottom-bar { background: white; padding: 8px 12px; display: flex; align-items: center; gap: 10px; border-top: 1px solid #ddd; }
        #msg-input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; background: #f4f4f5; font-size: 16px; }
        .icon-btn { font-size: 24px; color: var(--gray); cursor: pointer; }

        /* Панель эмодзи */
        #emoji-picker { display: none; background: #f4f4f5; padding: 10px; grid-template-columns: repeat(auto-fill, minmax(45px, 1fr)); gap: 5px; max-height: 180px; overflow-y: auto; }
        .emoji-item { font-size: 28px; text-align: center; cursor: pointer; }

        /* Окно записи кружочка */
        #circle-rec-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 2000; justify-content: center; align-items: center; flex-direction: column; }
        #rec-preview { width: 280px; height: 280px; border-radius: 50%; border: 4px solid white; object-fit: cover; transform: scaleX(-1); }
        .stop-btn { margin-top: 25px; padding: 15px 35px; border-radius: 30px; background: var(--red); color: white; border: none; font-weight: bold; font-size: 16px; cursor: pointer; box-shadow: 0 4px 15px rgba(255, 77, 77, 0.4); }
    </style>
</head>
<body>

    <audio id="notif-sound" src="https://www.soundjay.com/buttons/sounds/button-21.mp3"></audio>

    <div id="auth-screen" class="screen active">
        <div class="auth-card">
            <h2 style="color:var(--tg-blue)">Stitch-Gram</h2>
            <input type="text" id="username" placeholder="Твой ник">
            <button onclick="login()">Войти</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header><b>Контакты</b></header>
        <div class="search-area">
            <div class="search-box">
                <i class="fa-solid fa-magnifying-glass"></i>
                <input type="text" id="search-in" placeholder="Найти друга..." oninput="searchUsers()">
            </div>
        </div>
        <div id="search-results"></div>
        <div style="padding:10px 15px; font-size:12px; color:var(--tg-blue); font-weight:bold;">МОИ ДРУЗЬЯ</div>
        <div id="friends-list" style="flex:1; overflow-y:auto;"></div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left back-btn" onclick="closeChat()"></i>
            <div style="flex:1">
                <div id="h-name" style="font-weight:bold;">Чат</div>
                <div id="h-status" style="font-size:11px; opacity:0.8;">в сети</div>
            </div>
        </header>
        <div id="messages"></div>
        <div id="emoji-picker"></div>
        <div class="bottom-bar">
            <i class="fa-regular fa-face-smile icon-btn" onclick="toggleEmoji()"></i>
            <i class="fa-solid fa-circle-play icon-btn" style="color:var(--tg-blue)" onclick="startCircle()"></i>
            <input type="text" id="msg-input" placeholder="Сообщение..." oninput="isTyping()" onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane icon-btn" onclick="send()" style="color:var(--tg-blue)"></i>
        </div>
    </div>

    <div id="circle-rec-overlay">
        <video id="rec-preview" autoplay muted playsinline></video>
        <div id="rec-timer" style="color:white; margin-top:10px; font-weight:bold;">Идет запись...</div>
        <button class="stop-btn" id="stop-btn-action">ОСТАНОВИТЬ И ОТПРАВИТЬ</button>
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
    let mediaRecorder, stream, chunks = [];

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
            document.getElementById('h-status').innerText = s.val() ? 'печатает...' : 'в сети';
        });
        loadMessages();
    }

    /* РАБОЧИЙ КРУЖОЧЕК */
    async function startCircle() {
        const overlay = document.getElementById('circle-rec-overlay');
        const preview = document.getElementById('rec-preview');
        const stopBtn = document.getElementById('stop-btn-action');
        
        try {
            stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
            preview.srcObject = stream;
            overlay.style.display = 'flex';
            
            mediaRecorder = new MediaRecorder(stream);
            chunks = [];

            mediaRecorder.ondataavailable = e => { if(e.data.size > 0) chunks.push(e.data); };
            
            mediaRecorder.onstop = () => {
                const blob = new Blob(chunks, { type: 'video/webm' });
                const reader = new FileReader();
                reader.readAsDataURL(blob);
                reader.onloadend = () => {
                    db.ref('privates/' + chatID).push({
                        s: myName, type: 'circle', v: reader.result,
                        time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})
                    });
                };
                // Выключаем камеру
                stream.getTracks().forEach(track => track.stop());
                overlay.style.display = 'none';
            };

            mediaRecorder.start();
            
            stopBtn.onclick = () => {
                mediaRecorder.stop();
            };

        } catch (err) {
            alert("Нужен доступ к камере и микрофону!");
        }
    }

    function send() {
        const input = document.getElementById('msg-input');
        if(!input.value.trim()) return;
        db.ref('privates/' + chatID).push({
            s: myName, t: input.value, type: 'text',
            time: new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})
        });
        input.value = "";
    }

    function loadMessages() {
        db.ref('privates/' + chatID).on('value', snap => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            if(snap.val()) {
                Object.values(snap.val()).forEach(m => {
                    const div = document.createElement('div');
                    div.className = `msg ${m.s === myName ? 'sent' : 'received'}`;
                    if(m.type === 'circle') {
                        div.innerHTML = `<video class="circle-v" src="${m.v}" autoplay loop muted playsinline></video>
                                         <div style="font-size:10px; opacity:0.4; text-align:right;">${m.time}</div>`;
                    } else {
                        div.innerHTML = `${m.t} <div style="font-size:10px; opacity:0.4; text-align:right;">${m.time}</div>`;
                    }
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
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
        const list = ['😂','😍','😭','👍','🔥','❤️','😎','😊','🚀','👏','🤔','✨','✅','👋','🌚','🤡','💩','⚡️','🎉'];
        p.innerHTML = '';
        list.forEach(e => {
            const s = document.createElement('span');
            s.className = 'emoji-item';
            s.innerText = e;
            s.onclick = () => { document.getElementById('msg-input').value += e; document.getElementById('msg-input').focus(); };
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

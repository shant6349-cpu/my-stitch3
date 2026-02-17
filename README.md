<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch-Gram Classic</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        :root { --tg-blue: #248bcf; --tg-bg: #e7ebf0; --white: #ffffff; --gray: #8e8e93; --red: #ff4d4d; --green: #4cd964; }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: -apple-system, sans-serif; }
        body { background: var(--tg-bg); height: 100dvh; display: flex; justify-content: center; overflow: hidden; }

        .screen { position: fixed; inset: 0; display: none; background: white; z-index: 10; flex-direction: column; }
        .active { display: flex; }

        header { background: var(--tg-blue); color: white; padding: 12px 15px; display: flex; align-items: center; gap: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .back-btn { cursor: pointer; font-size: 20px; }

        /* Stories как в телеге */
        #stories-bar { display: flex; gap: 12px; padding: 10px 15px; background: white; border-bottom: 1px solid #eee; overflow-x: auto; scrollbar-width: none; }
        .story-node { flex-shrink: 0; width: 65px; text-align: center; font-size: 11px; cursor: pointer; }
        .story-ring { width: 60px; height: 60px; border-radius: 50%; border: 2.5px solid var(--tg-blue); padding: 2px; margin-bottom: 4px; }
        .story-ring.plus { border: 2px dashed #bbb; display: flex; align-items: center; justify-content: center; color: var(--tg-blue); font-size: 20px; }
        .av-text { width: 100%; height: 100%; border-radius: 50%; background: var(--tg-blue); color: white; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 20px; }

        /* Поиск и ID */
        .my-id-bar { background: #f0f7ff; padding: 8px 15px; font-size: 13px; color: var(--tg-blue); border-bottom: 1px solid #e0e0e0; font-weight: bold; }
        .search-area { padding: 10px; border-bottom: 1px solid #eee; }
        .search-box { background: #f2f2f7; border-radius: 10px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .search-box input { border: none; background: none; outline: none; width: 100%; font-size: 16px; }

        /* Сообщения */
        #messages { flex: 1; padding: 15px; overflow-y: auto; background: #96afc3 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); display: flex; flex-direction: column; gap: 8px; }
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 15px; background: white; box-shadow: 0 1px 2px rgba(0,0,0,0.1); }
        .msg.sent { align-self: flex-end; background: #effdde; }
        .circle-v { width: 220px; height: 220px; border-radius: 50%; object-fit: cover; border: 2px solid var(--tg-blue); }

        /* Нижняя панель - ВСЕ КНОПКИ ВЕРНУЛИСЬ */
        .bottom-bar { background: white; padding: 8px 12px; display: flex; align-items: center; gap: 10px; border-top: 1px solid #ddd; }
        #msg-input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; background: #f4f4f5; font-size: 16px; }
        .icon-btn { font-size: 24px; color: var(--gray); cursor: pointer; }

        /* Звонок */
        #call-screen { display: none; position: fixed; inset: 0; background: #1c1c1c; z-index: 5000; flex-direction: column; align-items: center; justify-content: center; color: white; }
        video#remote-v { width: 100%; height: 100%; object-fit: cover; }
        .call-tools { position: absolute; bottom: 50px; display: flex; gap: 30px; }
        .hangup { background: var(--red); width: 60px; height: 60px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 24px; border: none; color: white; }

        #rec-overlay, #story-view { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 3000; justify-content: center; align-items: center; flex-direction: column; }
    </style>
</head>
<body>

    <div id="auth-screen" class="screen active" style="justify-content:center; align-items:center; background:var(--tg-blue);">
        <div style="background:white; padding:30px; border-radius:20px; width:85%; text-align:center;">
            <h2 style="color:var(--tg-blue)">Stitch-Gram</h2>
            <input type="text" id="username" placeholder="Твой ник">
            <button onclick="login()" style="width:100%; padding:12px; background:var(--tg-blue); color:white; border:none; border-radius:10px; margin-top:10px;">Войти</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header><b>Stitch-Gram</b></header>
        <div class="my-id-bar" id="my-id-label">Мой ID: ...</div>
        
        <div id="stories-bar">
            <label class="story-node">
                <input type="file" accept="video/*" style="display:none;" onchange="uploadStory(this)">
                <div class="story-ring plus"><i class="fa-solid fa-plus"></i></div>
                <div>Сторис</div>
            </label>
            <div id="others-stories" style="display:flex; gap:12px;"></div>
        </div>

        <div class="search-area">
            <div class="search-box">
                <i class="fa-solid fa-magnifying-glass"></i>
                <input type="number" id="id-search" placeholder="Поиск по ID..." oninput="searchByID()">
            </div>
        </div>
        <div id="search-results" style="flex:1; overflow-y:auto;"></div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left back-btn" onclick="closeChat()"></i>
            <div style="flex:1"><b id="h-name">Чат</b></div>
            <i class="fa-solid fa-video" onclick="makeCall()" style="margin-right:15px; cursor:pointer;"></i>
            <i class="fa-solid fa-trash-can" onclick="clearChat()" style="cursor:pointer;"></i>
        </header>
        <div id="messages"></div>
        <div class="bottom-bar">
            <i class="fa-regular fa-face-smile icon-btn"></i>
            <i class="fa-solid fa-circle-play icon-btn" style="color:var(--tg-blue)" onclick="startCircle()"></i>
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') sendText()">
            <i id="voice-btn" class="fa-solid fa-microphone icon-btn" onclick="toggleVoice()"></i>
            <i class="fa-solid fa-paper-plane icon-btn" onclick="sendText()" style="color:var(--tg-blue)"></i>
        </div>
    </div>

    <div id="call-screen">
        <video id="remote-v" autoplay playsinline></video>
        <div class="call-tools">
            <button class="hangup" onclick="endCall()"><i class="fa-solid fa-phone-slash"></i></button>
        </div>
    </div>

    <div id="rec-overlay">
        <video id="rec-video" autoplay muted playsinline style="width:250px; height:250px; border-radius:50%; object-fit:cover; border:3px solid white;"></video>
        <button onclick="stopCircle()" style="margin-top:20px; padding:10px 20px; border-radius:20px; background:var(--red); color:white; border:none;">ОТПРАВИТЬ КРУЖОК</button>
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
    let myName = "", myID = "", chatPartner = "", chatID = "", mediaRecorder, chunks = [], isVoice = false;

    async function login() {
        myName = document.getElementById('username').value.trim();
        if(!myName) return;
        const snap = await db.ref('users/' + myName).once('value');
        if(snap.exists()) myID = snap.val().id;
        else { myID = Math.floor(1000 + Math.random() * 9000); await db.ref('users/' + myName).set({id: myID}); }
        document.getElementById('my-id-label').innerText = "Ник: " + myName + " | ID: " + myID;
        document.getElementById('auth-screen').classList.remove('active');
        document.getElementById('menu-screen').classList.add('active');
        loadStories();
    }

    function searchByID() {
        const val = document.getElementById('id-search').value;
        const res = document.getElementById('search-results');
        res.innerHTML = '';
        if(!val) return;
        db.ref('users').once('value', snap => {
            snap.forEach(u => {
                if(u.val().id && u.val().id.toString() === val) {
                    const div = document.createElement('div');
                    div.style = "padding:15px; border-bottom:1px solid #eee; font-weight:bold;";
                    div.innerText = u.key;
                    div.onclick = () => openChat(u.key);
                    res.appendChild(div);
                }
            });
        });
    }

    function openChat(p) {
        chatPartner = p;
        chatID = [myName, p].sort().join("_");
        document.getElementById('h-name').innerText = p;
        document.getElementById('chat-window').classList.add('active');
        db.ref('privates/' + chatID).on('value', snap => {
            const area = document.getElementById('messages'); area.innerHTML = '';
            snap.forEach(m => {
                const d = m.val();
                const div = document.createElement('div');
                div.className = `msg ${d.s === myName ? 'sent' : 'received'}`;
                if(d.type === 'circle') div.innerHTML = `<video class="circle-v" src="${d.v}" autoplay loop muted playsinline></video>`;
                else if(d.type === 'voice') div.innerHTML = `<audio controls src="${d.v}"></audio>`;
                else div.innerText = d.t;
                area.appendChild(div);
            });
            area.scrollTop = area.scrollHeight;
        });
    }

    function sendText() {
        const val = document.getElementById('msg-input').value;
        if(val) db.ref('privates/' + chatID).push({ s: myName, t: val, type: 'text' });
        document.getElementById('msg-input').value = "";
    }

    /* СТОРИС */
    function uploadStory(input) {
        const reader = new FileReader();
        reader.onload = (e) => db.ref('stories/' + myName).set({ v: e.target.result });
        reader.readAsDataURL(input.files[0]);
    }

    function loadStories() {
        db.ref('stories').on('value', snap => {
            const bar = document.getElementById('others-stories'); bar.innerHTML = '';
            snap.forEach(s => {
                if(s.key === myName) return;
                const div = document.createElement('div');
                div.className = 'story-node';
                div.innerHTML = `<div class="story-ring"><div class="av-text">${s.key[0]}</div></div><div>${s.key}</div>`;
                div.onclick = () => { /* Тут просто просмотр видео */ alert("Просмотр сторис от " + s.key); };
                bar.appendChild(div);
            });
        });
    }

    /* ЗВОНКИ (УПРОЩЕННО) */
    function makeCall() {
        document.getElementById('call-screen').style.display = 'flex';
        db.ref('calls/' + chatPartner).set({ from: myName, status: 'calling' });
    }
    function endCall() {
        document.getElementById('call-screen').style.display = 'none';
        db.ref('calls/' + chatPartner).remove();
    }

    /* КРУЖКИ */
    async function startCircle() {
        const stream = await navigator.mediaDevices.getUserMedia({video:true, audio:true});
        document.getElementById('rec-video').srcObject = stream;
        document.getElementById('rec-overlay').style.display = 'flex';
        mediaRecorder = new MediaRecorder(stream);
        chunks = [];
        mediaRecorder.ondataavailable = e => chunks.push(e.data);
        mediaRecorder.onstop = () => {
            const blob = new Blob(chunks, {type:'video/mp4'});
            const r = new FileReader();
            r.onload = () => db.ref('privates/' + chatID).push({ s: myName, type: 'circle', v: r.result });
            r.readAsDataURL(blob);
            stream.getTracks().forEach(t => t.stop());
            document.getElementById('rec-overlay').style.display = 'none';
        };
        mediaRecorder.start();
    }
    function stopCircle() { mediaRecorder.stop(); }

    function toggleVoice() { /* Логика ГС аналогична кружку */ }
    function closeChat() { document.getElementById('chat-window').classList.remove('active'); }
    function clearChat() { db.ref('privates/' + chatID).remove(); }
</script>
</body>
</html>

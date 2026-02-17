<!DOCTYPE html>
<html lang="hy">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Stitch-Gram Pro Edit</title>
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
        .header-actions { margin-left: auto; display: flex; gap: 15px; font-size: 18px; }

        #messages { flex: 1; padding: 15px; overflow-y: auto; background: #96afc3 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); display: flex; flex-direction: column; gap: 8px; }
        .msg { max-width: 80%; padding: 8px 12px; border-radius: 15px; font-size: 15px; background: white; box-shadow: 0 1px 2px rgba(0,0,0,0.1); position: relative; cursor: pointer; }
        .msg.sent { align-self: flex-end; background: #effdde; }
        
        .circle-v { width: 200px; height: 200px; border-radius: 50%; object-fit: cover; border: 2px solid var(--tg-blue); background: black; display: block; }

        .bottom-bar { background: white; padding: 8px 12px; display: flex; align-items: center; gap: 10px; border-top: 1px solid #ddd; }
        #msg-input { flex: 1; padding: 10px 15px; border-radius: 20px; border: 1px solid #ddd; outline: none; background: #f4f4f5; font-size: 16px; }

        #circle-rec-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 2000; justify-content: center; align-items: center; flex-direction: column; }
        #rec-preview { width: 280px; height: 280px; border-radius: 50%; border: 4px solid white; object-fit: cover; transform: scaleX(-1); }
        .stop-btn { margin-top: 25px; padding: 15px 35px; border-radius: 30px; background: var(--red); color: white; border: none; font-weight: bold; }
    </style>
</head>
<body>

    <div id="auth-screen" class="screen active">
        <div style="padding:40px; text-align:center;">
            <h2>Stitch-Gram</h2>
            <input type="text" id="username" placeholder="Մուտքանուն" style="width:100%; padding:12px; margin:20px 0; border-radius:10px; border:1px solid #ddd;">
            <button onclick="login()" style="width:100%; padding:12px; background:var(--tg-blue); color:white; border:none; border-radius:10px;">Մուտք</button>
        </div>
    </div>

    <div id="menu-screen" class="screen">
        <header><b>Կոնտակտներ</b></header>
        <div id="friends-list" style="flex:1; overflow-y:auto;">
             <div class="user-item" onclick="openChat('Մարիամ')" style="padding:15px; border-bottom:1px solid #eee; cursor:pointer;"><b>Մարիամ</b></div>
        </div>
    </div>

    <div id="chat-window" class="screen">
        <header>
            <i class="fa-solid fa-arrow-left back-btn" onclick="closeChat()"></i>
            <div style="flex:1">
                <div id="h-name" style="font-weight:bold;">Չատ</div>
                <div id="h-status" style="font-size:11px; opacity:0.8;">օնլայն</div>
            </div>
            <div class="header-actions">
                <i class="fa-solid fa-trash" onclick="deleteFullChat()" title="Ջնջել չատը"></i>
            </div>
        </header>
        <div id="messages"></div>
        <div class="bottom-bar">
            <i class="fa-solid fa-circle-play" style="color:var(--tg-blue); font-size:28px;" onclick="startCircle()"></i>
            <input type="text" id="msg-input" placeholder="Հաղորդագրություն..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()" style="color:var(--tg-blue); font-size:24px;"></i>
        </div>
    </div>

    <div id="circle-rec-overlay">
        <video id="rec-preview" autoplay muted playsinline></video>
        <button class="stop-btn" onclick="mediaRecorder.stop()">ԿԱՆԳՆԵՑՆԵԼ ԵՎ ՈՒՂԱՐԿԵԼ</button>
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
    let myName = "", chatPartner = "Մարիամ", chatID = "global_chat", editingMsgId = null;
    let mediaRecorder, stream, chunks = [];

    function login() {
        myName = document.getElementById('username').value.trim();
        if(myName) {
            document.getElementById('auth-screen').classList.remove('active');
            document.getElementById('menu-screen').classList.add('active');
        }
    }

    function openChat(p) {
        chatPartner = p;
        document.getElementById('chat-window').classList.add('active');
        loadMessages();
    }

    async function startCircle() {
        const overlay = document.getElementById('circle-rec-overlay');
        const preview = document.getElementById('rec-preview');
        stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
        preview.srcObject = stream;
        overlay.style.display = 'flex';
        mediaRecorder = new MediaRecorder(stream);
        chunks = [];
        mediaRecorder.ondataavailable = e => chunks.push(e.data);
        mediaRecorder.onstop = () => {
            const blob = new Blob(chunks, { type: 'video/mp4' });
            const reader = new FileReader();
            reader.readAsDataURL(blob);
            reader.onloadend = () => {
                db.ref('privates/' + chatID).push({
                    s: myName, type: 'circle', v: reader.result, time: new Date().toLocaleTimeString()
                });
            };
            stream.getTracks().forEach(t => t.stop());
            overlay.style.display = 'none';
        };
        mediaRecorder.start();
    }

    function send() {
        const input = document.getElementById('msg-input');
        const val = input.value.trim();
        if(!val) return;

        if(editingMsgId) {
            db.ref('privates/' + chatID + '/' + editingMsgId).update({ t: val, edited: true });
            editingMsgId = null;
        } else {
            db.ref('privates/' + chatID).push({
                s: myName, type: 'text', t: val, time: new Date().toLocaleTimeString()
            });
        }
        input.value = "";
    }

    function loadMessages() {
        db.ref('privates/' + chatID).on('value', snap => {
            const area = document.getElementById('messages');
            area.innerHTML = '';
            if(snap.val()) {
                Object.entries(snap.val()).forEach(([id, m]) => {
                    const isMe = m.s === myName;
                    const div = document.createElement('div');
                    div.className = `msg ${isMe ? 'sent' : 'received'}`;
                    
                    if(isMe && m.type === 'text') {
                        div.onclick = () => handleMsgAction(id, m.t);
                    }

                    if(m.type === 'circle') {
                        // Կարևոր fix վիդեոյի համար
                        div.innerHTML = `<video class="circle-v" src="${m.v}" autoplay loop muted playsinline></video>`;
                    } else {
                        div.innerHTML = `${m.t} ${m.edited ? '<small>(խմբագրված)</small>' : ''} <div style="font-size:10px; opacity:0.5; text-align:right;">${m.time}</div>`;
                    }
                    area.appendChild(div);
                });
                area.scrollTop = area.scrollHeight;
            }
        });
    }

    function handleMsgAction(id, text) {
        const action = confirm("Ընտրեք գործողությունը:\nOK - Խմբագրել\nCancel - Ջնջել");
        if(action) {
            editingMsgId = id;
            document.getElementById('msg-input').value = text;
            document.getElementById('msg-input').focus();
        } else {
            if(confirm("Ջնջե՞լ հաղորդագրությունը:")) {
                db.ref('privates/' + chatID + '/' + id).remove();
            }
        }
    }

    function deleteFullChat() {
        if(confirm("Զգուշացում! Դուք ուզում եք ջնջել ամբողջ չատի պատմությունը բոլորի մոտ:")) {
            db.ref('privates/' + chatID).remove();
        }
    }

    function closeChat() { document.getElementById('chat-window').classList.remove('active'); }
</script>
</body>
</html>

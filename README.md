<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #e7ebf0; font-family: -apple-system, sans-serif; }
        
        #auth-screen { position: fixed; inset: 0; background: white; z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 20px; }
        #chat-window { display: none; flex-direction: column; width: 100vw; height: 100vh; background: #c7d2d9 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); background-size: cover; }
        
        header { background: #3390ec; color: white; padding: 15px; display: flex; justify-content: space-between; align-items: center; width: 100%; }
        #messages { flex: 1; padding: 10px; overflow-y: auto; display: flex; flex-direction: column; gap: 8px; width: 100%; }
        
        .msg { max-width: 85%; padding: 10px; border-radius: 15px; font-size: 16px; box-shadow: 0 1px 2px rgba(0,0,0,0.1); word-wrap: break-word; }
        .msg.sent { align-self: flex-end; background: #effdde; border-bottom-right-radius: 2px; }
        .msg.received { align-self: flex-start; background: #fff; border-bottom-left-radius: 2px; }
        .msg-info { font-size: 11px; color: #888; margin-bottom: 3px; display: block; }

        .input-area { background: #fff; padding: 10px; display: flex; gap: 10px; align-items: center; border-top: 1px solid #ddd; width: 100%; }
        .input-area input { flex: 1; padding: 12px; border: none; background: #f1f1f1; border-radius: 25px; outline: none; font-size: 16px; }
    </style>
</head>
<body>
    <div id="auth-screen">
        <div style="width: 100%; max-width: 300px; text-align: center;">
            <h1 style="color: #3390ec; margin-bottom: 20px;">Stitch</h1>
            <input type="text" id="username" placeholder="Твой ник..." style="width:100%; padding:15px; border:1px solid #ddd; border-radius:10px; margin-bottom:10px;">
            <button onclick="login()" style="width:100%; padding:15px; background:#3390ec; color:white; border:none; border-radius:10px; font-weight:bold;">Войти</button>
        </div>
    </div>

    <div id="chat-window">
        <header><b>Stitch Online</b><i class="fa-solid fa-sync" onclick="location.reload()"></i></header>
        <div id="messages"></div>
        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()" style="color:#3390ec; font-size:24px;"></i>
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

        function loadMessages() {
            db.ref('messages').on('value', (snapshot) => {
                const area = document.getElementById('messages');
                area.innerHTML = '';
                const data = snapshot.val();
                if (data) {
                    Object.values(data).forEach(m => {
                        const div = document.createElement('div');
                        div.className = `msg ${m.user === myName ? 'sent' : 'received'}`;
                        div.innerHTML = `<span class="msg-info"><b>${m.user}</b></span>${m.text}`;
                        area.appendChild(div);
                    });
                    area.scrollTop = area.scrollHeight;
                }
            });
        }
    </script>
</body>
</html>

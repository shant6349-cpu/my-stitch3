<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Stitch Online</title>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-database-compat.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        
        body { 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; 
            background: #e7ebf0; 
            height: 100vh; 
            width: 100vw;
            display: flex; 
            flex-direction: column; 
            overflow: hidden;
        }

        /* Экран входа */
        #auth-screen { 
            position: fixed; 
            inset: 0; 
            background: white; 
            z-index: 1000; 
            display: flex; 
            flex-direction: column; 
            align-items: center; 
            justify-content: center; 
            padding: 20px;
        }
        .auth-box { width: 100%; max-width: 350px; text-align: center; }
        .auth-box input { width: 100%; padding: 15px; margin: 10px 0; border: 1px solid #ddd; border-radius: 12px; font-size: 16px; outline: none; }
        .auth-box button { width: 100%; padding: 15px; background: #3390ec; color: white; border: none; border-radius: 12px; font-weight: bold; font-size: 16px; cursor: pointer; }

        /* Окно чата на весь экран */
        #chat-window { 
            display: none; 
            flex-direction: column; 
            width: 100%; 
            height: 100vh; 
            background: #c7d2d9 url('https://user-images.githubusercontent.com/15075759/28719144-86dc0f70-73b1-11e7-911d-60d70fcded21.png'); 
            background-size: cover; 
        }

        header { 
            background: #3390ec; 
            color: white; 
            padding: 12px 20px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        /* Сообщения стали ШИРОКИМИ */
        #messages { 
            flex: 1; 
            padding: 15px; 
            overflow-y: auto; 
            display: flex; 
            flex-direction: column; 
            gap: 10px; 
            width: 100%;
        }

        .msg { 
            max-width: 85%; /* Теперь сообщения занимают больше места */
            padding: 10px 14px; 
            border-radius: 15px; 
            font-size: 16px; 
            line-height: 1.4;
            box-shadow: 0 1px 2px rgba(0,0,0,0.15); 
            word-wrap: break-word; 
            position: relative; 
        }

        .msg.sent { 
            align-self: flex-end; 
            background: #effdde; 
            border-bottom-right-radius: 4px; 
        }

        .msg.received { 
            align-self: flex-start; 
            background: #ffffff; 
            border-bottom-left-radius: 4px; 
        }

        .msg-info { 
            font-size: 11px; 
            color: #888; 
            margin-bottom: 4px; 
            display: block;
        }

        .msg-time {
            font-size: 10px;
            color: #999;
            float: right;
            margin-top: 5px;
            margin-left: 10px;
        }

        /* Поле ввода во всю ширину */
        .input-area { 
            background: #fff; 
            padding: 10px 15px; 
            display: flex; 
            gap: 12px; 
            align-items: center; 
            border-top: 1px solid #ddd;
        }
        .input-area input { 
            flex: 1; 
            padding: 12px 18px; 
            border: 1px solid #eee; 
            background: #f1f1f1; 
            border-radius: 25px; 
            font-size: 16px; 
            outline: none;
        }
        .input-area i { 
            color: #3390ec; 
            font-size: 26px; 
            cursor: pointer; 
        }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-box">
            <h1 style="color: #3390ec; font-size: 32px; margin-bottom: 10px;">Stitch</h1>
            <p style="color: #888; margin-bottom: 20px;">Онлайн мессенджер</p>
            <input type="text" id="username" placeholder="Ваше имя...">
            <button onclick="login()">Войти в чат</button>
        </div>
    </div>

    <div id="chat-window">
        <header>
            <div style="display: flex; align-items: center; gap: 10px;">
                <div style="width: 35px; height: 35px; background: #fff; border-radius: 50%; color: #3390ec; display: flex; align-items: center; justify-content: center; font-weight: bold;">S</div>
                <b id="user-display">Чат</b>
            </div>
            <i class="fa-solid fa-arrow-right-from-bracket" onclick="location.reload()"></i>
        </header>
        
        <div id="messages"></div>

        <div class="input-area">
            <input type="text" id="msg-input" placeholder="Сообщение..." onkeypress="if(event.key==='Enter') send()">
            <i class="fa-solid fa-paper-plane" onclick="send()"></i>
        </div>
    </div>

<script>
    // ТВОИ ДАННЫЕ С ФОТО
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
        if (myName.length < 2) return alert("Введите имя!");
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('chat-window').style.display = 'flex';
        document.getElementById('user-display').innerText = myName;
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
                    div.innerHTML = `
                        <span class="msg-info"><b>${m.user}</b></span>
                        ${m.text}
                        <span class="msg-time">${m.time}</span>
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

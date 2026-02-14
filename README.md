<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Kawaii Edition</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { 
            --pink: #ffcce5; 
            --purple: #e2d1f9;
            --blue: #d1e9ff;
            --accent: #ff85a1;
            --text: #5a4a5a;
            --white: #ffffff;
        }

        body { 
            font-family: 'Segoe UI', Roboto, sans-serif; 
            margin: 0; 
            background: linear-gradient(135deg, var(--pink), var(--purple), var(--blue));
            background-attachment: fixed;
            color: var(--text);
            padding-bottom: 100px;
        }

        /* --- ШАПКА --- */
        header { 
            background: rgba(255, 255, 255, 0.8); 
            backdrop-filter: blur(10px);
            padding: 15px 20px; 
            border-bottom: 2px solid var(--pink); 
            display: flex; justify-content: space-between; align-items: center; 
            position: sticky; top: 0; z-index: 1000;
        }
        header .logo { font-weight: 900; font-size: 24px; color: var(--accent); letter-spacing: 1px; }

        /* --- КАРТОЧКА ПОСТА --- */
        .view-content { max-width: 500px; margin: 0 auto; padding: 15px; }
        
        .post { 
            background: var(--white); border-radius: 30px; 
            margin-bottom: 25px; overflow: hidden; 
            box-shadow: 0 10px 25px rgba(255, 133, 161, 0.15);
            border: 2px solid #fff;
            animation: bounceIn 0.6s ease;
        }
        @keyframes bounceIn {
            0% { opacity: 0; transform: translateY(20px); }
            100% { opacity: 1; transform: translateY(0); }
        }

        .post-header { padding: 12px 18px; display: flex; align-items: center; gap: 12px; font-weight: bold; }
        .user-ava { width: 40px; height: 40px; border-radius: 50%; border: 2px solid var(--pink); object-fit: cover; }
        .post-img { width: 100%; display: block; max-height: 500px; object-fit: cover; }
        
        .post-footer { padding: 15px 20px; }
        .post-icons { display: flex; gap: 20px; font-size: 24px; margin-bottom: 10px; color: var(--accent); }
        .fa-heart.active { color: #ff3040; animation: heartBeat 0.3s ease; }
        @keyframes heartBeat { 50% { transform: scale(1.3); } }

        /* --- НАВИГАЦИЯ --- */
        nav { 
            position: fixed; bottom: 20px; left: 20px; right: 20px;
            background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px);
            border-radius: 35px; display: flex; justify-content: space-around; 
            padding: 12px 0; z-index: 1000; box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border: 2px solid var(--pink);
        }
        .nav-link { font-size: 24px; color: #d1b2d1; cursor: pointer; transition: 0.3s; }
        .nav-link.active { color: var(--accent); transform: scale(1.2); }

        /* --- МОДАЛКА --- */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 5000; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
        .modal-content { background: white; padding: 25px; border-radius: 30px; width: 90%; max-width: 380px; border: 3px solid var(--pink); }
        input, textarea, select { width: 100%; padding: 12px; margin-bottom: 15px; border: 2px solid var(--purple); border-radius: 15px; box-sizing: border-box; outline: none; font-family: inherit; }
        .btn-action { background: var(--accent); color: white; border: none; padding: 14px; border-radius: 20px; width: 100%; font-weight: bold; cursor: pointer; transition: 0.3s; }
        .btn-action:hover { opacity: 0.8; transform: scale(0.98); }

        /* --- ЭКРАН ВХОДА --- */
        #auth-screen { position: fixed; inset: 0; background: linear-gradient(var(--pink), var(--purple)); z-index: 9999; display: flex; align-items: center; justify-content: center; }
        .auth-card { background: white; padding: 40px; border-radius: 35px; text-align: center; width: 85%; max-width: 350px; border: 4px solid var(--accent); box-shadow: 0 20px 40px rgba(0,0,0,0.1); }

        /* --- ПРОФИЛЬ --- */
        .profile-top { padding: 30px 20px; text-align: center; }
        .p-avatar { width: 100px; height: 100px; border-radius: 50%; border: 4px solid var(--white); box-shadow: 0 5px 15px var(--pink); margin-bottom: 10px; object-fit: cover; }
        .photo-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 5px; padding: 5px; }
        .photo-grid div { aspect-ratio: 1/1; overflow: hidden; border-radius: 10px; background: #eee; }
        .photo-grid img, .photo-grid video { width: 100%; height: 100%; object-fit: cover; }

        .comments-section { font-size: 13px; margin-top: 10px; padding-top: 10px; border-top: 1px solid #fef0f5; }
        .comment { margin-bottom: 4px; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <h1 style="color: var(--accent); margin: 0 0 10px 0;">StitchGram</h1>
            <p style="margin-bottom: 20px; font-size: 14px;">Создай свой уютный мир ✨</p>
            <input type="text" id="reg-user" placeholder="Как тебя зовут?">
            <p style="font-size: 12px; color: gray;">Фото профиля:</p>
            <input type="file" id="reg-file" accept="image/*" onchange="previewRegAva(this)">
            <img id="reg-ava-pre" style="width:80px; height:80px; border-radius:50%; display:none; margin: 10px auto; object-fit:cover; border:3px solid var(--pink);">
            <button class="btn-action" onclick="finishReg()">Войти ✨</button>
        </div>
    </div>

    <div id="app-container" style="display:none;">
        <header>
            <div class="logo">STITCHGRAM</div>
            <div style="display: flex; gap: 15px; color: var(--accent); font-size: 20px;">
                <i class="fa-solid fa-heart"></i>
                <i class="fa-solid fa-paper-plane"></i>
            </div>
        </header>

        <div id="home-view" class="view-content">
            <div id="feed-items"></div>
        </div>

        <div id="shorts-view" style="display:none; padding-bottom: 100px;">
            </div>

        <div id="profile-view" class="view-content" style="display:none;">
            <div class="profile-top">
                <img id="p-avatar-img" class="p-avatar" src="">
                <h2 id="p-username-txt" style="margin: 5px 0;"></h2>
                <button onclick="logout()" style="border:none; background:rgba(255,255,255,0.8); padding:5px 15px; border-radius:15px; font-size:12px; cursor:pointer;">Выйти 🚪</button>
            </div>
            <div class="photo-grid" id="p-grid-items"></div>
        </div>

        <nav>
            <i class="fa-solid fa-house nav-link active" onclick="switchTab('home', this)"></i>
            <i class="fa-solid fa-clapperboard nav-link" onclick="switchTab('shorts', this)"></i>
            <i class="fa-solid fa-circle-plus nav-link" onclick="openUploadModal()" style="font-size: 32px; color: var(--accent);"></i>
            <i class="fa-solid fa-face-smile nav-link" onclick="switchTab('profile', this)"></i>
        </nav>
    </div>

    <div id="upload-modal" class="modal">
        <div class="modal-content">
            <h3 style="text-align: center; color: var(--accent); margin-top: 0;">Создать магию ✨</h3>
            <select id="upload-type">
                <option value="post">Фото-пост</option>
                <option value="short">Shorts (Видео)</option>
            </select>
            <input type="file" id="upload-file" accept="image/*,video/*">
            <textarea id="upload-desc" placeholder="Напиши что-нибудь милое..."></textarea>
            <button class="btn-action" id="upload-btn" onclick="processUpload()">Поделиться!</button>
            <button onclick="closeUploadModal()" style="border:none; background:none; width:100%; margin-top:10px; color:gray; cursor:pointer;">Отмена</button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, query, orderBy, onSnapshot, updateDoc, doc, arrayUnion } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAHxAc0XUrq4N5nuH0UnD66PAnUXRU_DJQ",
            authDomain: "myprogram-1b6b0.firebaseapp.com",
            projectId: "myprogram-1b6b0",
            appId: "1:678957156444:web:6ac76957ea7e0ba2283e99"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        let me = JSON.parse(localStorage.getItem('sg_me'));
        let tempAva = "";

        window.previewRegAva = (i) => {
            const r = new FileReader();
            r.onload = e => { 
                tempAva = e.target.result; 
                document.getElementById('reg-ava-pre').src = e.target.result; 
                document.getElementById('reg-ava-pre').style.display='block'; 
            };
            r.readAsDataURL(i.files[0]);
        };

        window.finishReg = () => {
            const name = document.getElementById('reg-user').value.trim();
            if(!name) return alert("Введи имя, солнышко!");
            me = { name, ava: tempAva || 'https://via.placeholder.com/150' };
            localStorage.setItem('sg_me', JSON.stringify(me));
            location.reload();
        };

        window.switchTab = (tab, el) => {
            document.querySelectorAll('.view-content, #shorts-view').forEach(v => v.style.display = 'none');
            document.querySelectorAll('.nav-link').forEach(l => l.classList.remove('active'));
            document.getElementById(tab + '-view').style.display = 'block';
            el.classList.add('active');
            if(tab === 'profile') renderProfile();
        };

        window.openUploadModal = () => document.getElementById('upload-modal').style.display = 'flex';
        window.closeUploadModal = () => document.getElementById('upload-modal').style.display = 'none';

        window.processUpload = async () => {
            const type = document.getElementById('upload-type').value;
            const file = document.getElementById('upload-file').files[0];
            const desc = document.getElementById('upload-desc').value;
            if(!file) return alert("Выбери файл!");

            const btn = document.getElementById('upload-btn');
            btn.disabled = true;
            btn.innerText = "Магия колдуется...";

            const r = new FileReader();
            r.onload = async e => {
                try {
                    await addDoc(collection(db, type === 'post' ? "posts" : "shorts"), {
                        author: me.name,
                        ava: me.ava,
                        content: e.target.result,
                        text: desc,
                        likedUsers: [],
                        comments: [],
                        time: Date.now()
                    });
                    closeUploadModal();
                    btn.disabled = false;
                    btn.innerText = "Поделиться!";
                } catch (err) {
                    alert("Ошибка! Проверь правила в Firebase.");
                    btn.disabled = false;
                }
            };
            r.readAsDataURL(file);
        };

        window.toggleLike = async (id, likedUsers = []) => {
            const postRef = doc(db, "posts", id);
            const isLiked = likedUsers.includes(me.name);
            await updateDoc(postRef, {
                likedUsers: isLiked ? likedUsers.filter(u => u !== me.name) : arrayUnion(me.name)
            });
        };

        window.addComment = async (id) => {
            const input = document.getElementById('comm-in-' + id);
            if(!input.value.trim()) return;
            await updateDoc(doc(db, "posts", id), {
                comments: arrayUnion({ user: me.name, text: input.value })
            });
            input.value = "";
        };

        if(me) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-container').style.display = 'block';
            document.getElementById('home-view').style.display = 'block';

            // Лента
            onSnapshot(query(collection(db, "posts"), orderBy("time", "desc")), (s) => {
                document.getElementById('feed-items').innerHTML = s.docs.map(d => {
                    const p = d.data();
                    const isLiked = p.likedUsers?.includes(me.name);
                    return `
                    <div class="post">
                        <div class="post-header"><img src="${p.ava}" class="user-ava"> ${p.author}</div>
                        <img src="${p.content}" class="post-img" ondblclick="toggleLike('${d.id}', ${JSON.stringify(p.likedUsers || [])})">
                        <div class="post-footer">
                            <div class="post-icons">
                                <i class="fa-solid fa-heart ${isLiked ? 'active' : ''}" onclick="toggleLike('${d.id}', ${JSON.stringify(p.likedUsers || [])})"></i>
                                <i class="fa-regular fa-comment"></i>
                            </div>
                            <div style="font-size: 14px;"><strong>${p.author}</strong> ${p.text}</div>
                            <div class="comments-section">
                                ${p.comments?.map(c => `<div class="comment"><b>${c.user}</b> ${c.text}</div>`).join('') || ''}
                                <div style="display:flex; gap:5px; margin-top:8px;">
                                    <input type="text" id="comm-in-${d.id}" placeholder="Милый коммент..." style="margin:0; padding:5px 10px; font-size:12px; border-radius:10px;">
                                    <button onclick="addComment('${d.id}')" style="border:none; background:none; color:var(--accent); font-weight:bold; cursor:pointer;">ОК</button>
                                </div>
                            </div>
                        </div>
                    </div>`;
                }).join('');
            });

            // Shorts
            onSnapshot(query(collection(db, "shorts"), orderBy("time", "desc")), (s) => {
                document.getElementById('shorts-view').innerHTML = s.docs.map(d => {
                    const v = d.data();
                    return `
                    <div class="post" style="max-width:350px; margin: 20px auto;">
                        <div class="post-header"><img src="${v.ava}" class="user-ava"> ${v.author}</div>
                        <video src="${v.content}" style="width:100%;" loop autoplay muted onclick="this.paused?this.play():this.pause()"></video>
                        <div class="post-footer">
                            <div style="font-size: 14px;">${v.text}</div>
                        </div>
                    </div>`;
                }).join('');
            });
        }

        window.renderProfile = () => {
            document.getElementById('p-avatar-img').src = me.ava;
            document.getElementById('p-username-txt').innerText = me.name;
            onSnapshot(query(collection(db, "posts"), orderBy("time", "desc")), (s) => {
                document.getElementById('p-grid-items').innerHTML = s.docs
                    .filter(d => d.data().author === me.name)
                    .map(d => `<div><img src="${d.data().content}"></div>`).join('');
            });
        };

        window.logout = () => { localStorage.clear(); location.reload(); };
    </script>
</body>
</html>

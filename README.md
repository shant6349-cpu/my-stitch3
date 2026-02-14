<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Kawaii + Stories</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { 
            --pink: #ffcce5; --purple: #e2d1f9; --blue: #d1e9ff;
            --accent: #ff85a1; --text: #5a4a5a; --white: #ffffff;
            --insta-grad: linear-gradient(45deg, #f09433 0%, #e6683c 25%, #dc2743 50%, #cc2366 75%, #bc1888 100%);
        }

        body { 
            font-family: 'Segoe UI', Roboto, sans-serif; margin: 0; 
            background: linear-gradient(135deg, var(--pink), var(--purple), var(--blue));
            background-attachment: fixed; color: var(--text); padding-bottom: 100px;
        }

        header { 
            background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px);
            padding: 15px 20px; border-bottom: 2px solid var(--pink); 
            display: flex; justify-content: space-between; align-items: center; 
            position: sticky; top: 0; z-index: 1000;
        }
        header .logo { font-weight: 900; font-size: 24px; color: var(--accent); letter-spacing: 1px; }

        /* --- STORIES BLOCK --- */
        .stories-container {
            display: flex; gap: 15px; padding: 15px; overflow-x: auto;
            background: rgba(255, 255, 255, 0.3); margin-bottom: 10px;
            scrollbar-width: none; /* Hide scrollbar */
        }
        .stories-container::-webkit-scrollbar { display: none; }
        
        .story-circle {
            min-width: 65px; height: 65px; border-radius: 50%;
            padding: 3px; background: var(--insta-grad);
            display: flex; align-items: center; justify-content: center;
            cursor: pointer; transition: 0.3s;
        }
        .story-circle img {
            width: 100%; height: 100%; border-radius: 50%;
            object-fit: cover; border: 3px solid white;
        }
        .story-name { font-size: 10px; text-align: center; margin-top: 5px; font-weight: bold; color: var(--text); }

        /* --- POSTS --- */
        .view-content { max-width: 500px; margin: 0 auto; padding: 10px; }
        .post { 
            background: var(--white); border-radius: 30px; margin-bottom: 25px; 
            overflow: hidden; box-shadow: 0 10px 25px rgba(255, 133, 161, 0.15);
            border: 2px solid #fff; animation: bounceIn 0.6s ease;
        }
        @keyframes bounceIn { 0% { opacity: 0; transform: translateY(20px); } 100% { opacity: 1; transform: translateY(0); } }
        .post-header { padding: 12px 18px; display: flex; align-items: center; gap: 12px; font-weight: bold; }
        .user-ava { width: 40px; height: 40px; border-radius: 50%; border: 2px solid var(--pink); object-fit: cover; }
        .post-img { width: 100%; display: block; max-height: 500px; object-fit: cover; }
        .post-footer { padding: 15px 20px; }
        .post-icons { display: flex; gap: 20px; font-size: 24px; margin-bottom: 10px; color: var(--accent); }
        .fa-heart.active { color: #ff3040; animation: heartBeat 0.3s ease; }
        @keyframes heartBeat { 50% { transform: scale(1.3); } }

        /* --- NAV --- */
        nav { 
            position: fixed; bottom: 20px; left: 20px; right: 20px;
            background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px);
            border-radius: 35px; display: flex; justify-content: space-around; 
            padding: 12px 0; z-index: 1000; box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border: 2px solid var(--pink);
        }
        .nav-link { font-size: 24px; color: #d1b2d1; cursor: pointer; transition: 0.3s; }
        .nav-link.active { color: var(--accent); transform: scale(1.2); }

        /* --- MODAL --- */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 5000; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal-content { background: white; padding: 25px; border-radius: 30px; width: 90%; max-width: 380px; border: 3px solid var(--pink); }
        input, textarea, select { width: 100%; padding: 12px; margin-bottom: 15px; border: 2px solid var(--purple); border-radius: 15px; box-sizing: border-box; outline: none; }
        .btn-action { background: var(--accent); color: white; border: none; padding: 14px; border-radius: 20px; width: 100%; font-weight: bold; cursor: pointer; }

        /* FULL SCREEN STORY VIEW */
        #story-overlay { display: none; position: fixed; inset: 0; background: black; z-index: 6000; flex-direction: column; align-items: center; justify-content: center; }
        #story-overlay img { max-width: 100%; max-height: 80vh; border-radius: 15px; }
        #story-progress { width: 90%; height: 3px; background: #333; position: absolute; top: 20px; border-radius: 2px; }
        #story-bar { width: 0%; height: 100%; background: white; transition: 5s linear; }

        #auth-screen { position: fixed; inset: 0; background: linear-gradient(var(--pink), var(--purple)); z-index: 9999; display: flex; align-items: center; justify-content: center; }
        .auth-card { background: white; padding: 40px; border-radius: 35px; text-align: center; width: 85%; max-width: 350px; border: 4px solid var(--accent); }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <h1 style="color: var(--accent); margin-bottom: 5px;">StitchGram</h1>
            <p style="margin-bottom: 20px; font-size: 14px;">Создай свою историю ✨</p>
            <input type="text" id="reg-user" placeholder="Твое имя...">
            <input type="file" id="reg-file" accept="image/*" onchange="previewRegAva(this)">
            <img id="reg-ava-pre" style="width:70px; height:70px; border-radius:50%; display:none; margin: 10px auto; object-fit:cover; border:2px solid var(--pink);">
            <button class="btn-action" onclick="finishReg()">Войти ✨</button>
        </div>
    </div>

    <div id="app-container" style="display:none;">
        <header>
            <div class="logo">STITCHGRAM</div>
            <div style="color: var(--accent); font-size: 20px;"><i class="fa-solid fa-paper-plane"></i></div>
        </header>

        <div class="stories-container" id="stories-list">
            </div>

        <div id="home-view" class="view-content active-view">
            <div id="feed-items"></div>
        </div>

        <div id="shorts-view" style="display:none;"></div>

        <div id="profile-view" class="view-content" style="display:none;">
            <div style="text-align:center; padding: 20px;">
                <img id="p-avatar-img" style="width:80px; height:80px; border-radius:50%; border:3px solid white; box-shadow: 0 5px 15px var(--pink);">
                <h2 id="p-username-txt"></h2>
                <button onclick="logout()" style="border:none; background:#eee; padding:5px 15px; border-radius:15px; font-size:12px;">Выйти</button>
            </div>
            <div id="p-grid-items" style="display:grid; grid-template-columns:1fr 1fr 1fr; gap:5px;"></div>
        </div>

        <nav>
            <i class="fa-solid fa-house nav-link active" onclick="switchTab('home', this)"></i>
            <i class="fa-solid fa-clapperboard nav-link" onclick="switchTab('shorts', this)"></i>
            <i class="fa-solid fa-plus-circle nav-link" onclick="openUploadModal()" style="font-size: 30px; color: var(--accent);"></i>
            <i class="fa-solid fa-user nav-link" onclick="switchTab('profile', this)"></i>
        </nav>
    </div>

    <div id="story-overlay" onclick="closeStory()">
        <div id="story-progress"><div id="story-bar"></div></div>
        <img id="story-img" src="">
        <p id="story-author" style="color:white; margin-top:15px; font-weight:bold;"></p>
    </div>

    <div id="upload-modal" class="modal">
        <div class="modal-content">
            <h3 style="text-align: center; color: var(--accent);">Поделиться моментом ✨</h3>
            <select id="upload-type">
                <option value="post">Обычный пост</option>
                <option value="story">История (Stories)</option>
                <option value="short">Shorts (Видео)</option>
            </select>
            <input type="file" id="upload-file" accept="image/*,video/*">
            <textarea id="upload-desc" placeholder="Описание..."></textarea>
            <button class="btn-action" id="upload-btn" onclick="processUpload()">Опубликовать!</button>
            <button onclick="closeUploadModal()" style="border:none; background:none; width:100%; margin-top:10px; color:gray;">Отмена</button>
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
            r.onload = e => { tempAva = e.target.result; document.getElementById('reg-ava-pre').src = e.target.result; document.getElementById('reg-ava-pre').style.display='block'; };
            r.readAsDataURL(i.files[0]);
        };

        window.finishReg = () => {
            const name = document.getElementById('reg-user').value;
            if(!name) return alert("Введите имя");
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
            if(!file) return alert("Файл!");

            const r = new FileReader();
            r.onload = async e => {
                await addDoc(collection(db, type === 'post' ? "posts" : (type === 'story' ? "stories" : "shorts")), {
                    author: me.name, ava: me.ava, content: e.target.result, text: desc,
                    likedUsers: [], comments: [], time: Date.now()
                });
                closeUploadModal();
            };
            r.readAsDataURL(file);
        };

        // STORY LOGIC
        window.viewStory = (img, author) => {
            document.getElementById('story-overlay').style.display = 'flex';
            document.getElementById('story-img').src = img;
            document.getElementById('story-author').innerText = "@" + author;
            document.getElementById('story-bar').style.width = '100%';
            setTimeout(closeStory, 5000);
        };
        window.closeStory = () => {
            document.getElementById('story-overlay').style.display = 'none';
            document.getElementById('story-bar').style.width = '0%';
        };

        if(me) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-container').style.display = 'block';

            // Живые Stories
            onSnapshot(query(collection(db, "stories"), orderBy("time", "desc")), (s) => {
                const list = document.getElementById('stories-list');
                list.innerHTML = `<div style="text-align:center;"><div class="story-circle" style="background:#eee" onclick="openUploadModal()"><img src="${me.ava}"></div><div class="story-name">Ваша</div></div>`;
                list.innerHTML += s.docs.map(d => {
                    const st = d.data();
                    return `<div><div class="story-circle" onclick="viewStory('${st.content}', '${st.author}')"><img src="${st.ava}"></div><div class="story-name">${st.author}</div></div>`;
                }).join('');
            });

            // Посты (Лента)
            onSnapshot(query(collection(db, "posts"), orderBy("time", "desc")), (s) => {
                document.getElementById('feed-items').innerHTML = s.docs.map(d => {
                    const p = d.data();
                    const isLiked = p.likedUsers?.includes(me.name);
                    return `
                    <div class="post">
                        <div class="post-header"><img src="${p.ava}" class="user-ava"> ${p.author}</div>
                        <img src="${p.content}" class="post-img" ondblclick="toggleLike('${d.id}', ${JSON.stringify(p.likedUsers || [])})">
                        <div class="post-footer">
                            <div class="post-icons"><i class="fa-solid fa-heart ${isLiked ? 'active' : ''}"></i><i class="fa-regular fa-comment"></i></div>
                            <strong>${p.author}</strong> ${p.text}
                        </div>
                    </div>`;
                }).join('');
            });
        }

        window.renderProfile = () => {
            document.getElementById('p-avatar-img').src = me.ava;
            document.getElementById('p-username-txt').innerText = me.name;
        };
        window.logout = () => { localStorage.clear(); location.reload(); };
    </script>
</body>
</html>

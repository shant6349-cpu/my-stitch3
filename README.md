<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Ultimate OS</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --fb-blue: #0866ff; --inst-pink: #e1306c; --bg: #f0f2f5; --border: #dbdbdb; --white: #ffffff; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; margin: 0; background: var(--bg); color: #1c1e21; overflow-x: hidden; }

        /* --- AUTH --- */
        #auth-screen { position: fixed; inset: 0; background: var(--white); z-index: 9999; display: flex; align-items: center; justify-content: center; }
        .auth-card { width: 90%; max-width: 400px; padding: 40px; border-radius: 15px; box-shadow: 0 15px 35px rgba(0,0,0,0.1); text-align: center; border: 1px solid var(--border); }
        .auth-card h1 { font-style: italic; font-size: 45px; color: var(--fb-blue); margin-bottom: 10px; }

        /* --- UI ELEMENTS --- */
        input, textarea, select { width: 100%; padding: 14px; margin-bottom: 15px; border: 1px solid var(--border); border-radius: 8px; font-size: 16px; box-sizing: border-box; background: #f5f6f7; }
        .btn-action { background: var(--fb-blue); color: white; border: none; padding: 15px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 18px; cursor: pointer; }
        header { background: var(--white); padding: 10px 20px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; z-index: 1000; }
        .view-content { max-width: 600px; margin: 0 auto; padding-bottom: 80px; display: none; }
        .active-view { display: block; }

        /* --- POSTS --- */
        .post { background: var(--white); border: 1px solid var(--border); border-radius: 10px; margin: 20px 0; overflow: hidden; }
        .post-header { padding: 12px; display: flex; align-items: center; gap: 10px; font-weight: 600; }
        .user-ava { width: 36px; height: 36px; border-radius: 50%; object-fit: cover; }
        .post-img { width: 100%; display: block; max-height: 600px; object-fit: contain; background: #000; }
        .post-footer { padding: 15px; }
        .post-icons { display: flex; gap: 20px; font-size: 24px; margin-bottom: 10px; }
        .fa-heart.active { color: #ff3040; animation: pulse 0.3s; }
        @keyframes pulse { 50% {transform: scale(1.3);} }

        /* --- COMMENTS --- */
        .comments-section { border-top: 1px solid #f0f0f0; padding-top: 10px; margin-top: 10px; font-size: 14px; }
        .comment { margin-bottom: 5px; }
        .comment-input-area { display: flex; gap: 10px; margin-top: 10px; }
        .comment-input-area input { margin-bottom: 0; padding: 8px; }

        /* --- SHORTS --- */
        #shorts-view { background: #000; height: calc(100vh - 60px); overflow-y: scroll; scroll-snap-type: y mandatory; display: none; }
        .short-video-container { height: 100%; scroll-snap-align: start; position: relative; display: flex; align-items: center; justify-content: center; }
        .short-video { width: 100%; height: 100%; object-fit: cover; }
        .short-info { position: absolute; bottom: 40px; left: 15px; color: white; text-shadow: 1px 1px 5px rgba(0,0,0,0.8); }

        /* --- PROFILE --- */
        .profile-top { padding: 40px 20px; background: white; text-align: center; border-bottom: 1px solid var(--border); }
        .p-avatar { width: 100px; height: 100px; border-radius: 50%; object-fit: cover; border: 3px solid var(--fb-blue); }
        .photo-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 2px; padding: 2px; }
        .photo-grid img { width: 100%; aspect-ratio: 1/1; object-fit: cover; }

        /* --- NAV --- */
        nav { position: fixed; bottom: 0; width: 100%; background: white; border-top: 1px solid var(--border); display: flex; justify-content: space-around; padding: 12px 0; z-index: 1000; }
        .nav-link { font-size: 24px; color: #65676b; cursor: pointer; }
        .nav-link.active { color: var(--fb-blue); }

        /* --- MODAL --- */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 5000; align-items: center; justify-content: center; }
        .modal-content { background: white; padding: 25px; border-radius: 15px; width: 90%; max-width: 400px; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <h1>StitchGram</h1>
            <input type="text" id="reg-user" placeholder="Имя пользователя">
            <p style="font-size: 12px;">Выберите фото профиля:</p>
            <input type="file" id="reg-file" accept="image/*" onchange="previewRegAva(this)">
            <img id="reg-ava-pre" style="width:80px; height:80px; border-radius:50%; display:none; margin: 10px auto; object-fit:cover;">
            <button class="btn-action" onclick="finishReg()">Войти</button>
        </div>
    </div>

    <div id="app-container" style="display:none;">
        <header>
            <div style="font-weight: 900; font-size: 24px; color: var(--fb-blue);">StitchGram</div>
            <div style="display: flex; gap: 20px;"><i class="fa-solid fa-heart"></i><i class="fa-solid fa-paper-plane"></i></div>
        </header>

        <div id="home-view" class="view-content active-view">
            <div id="feed-items"></div>
        </div>

        <div id="shorts-view"></div>

        <div id="profile-view" class="view-content">
            <div class="profile-top">
                <img id="p-avatar-img" class="p-avatar" src="">
                <h2 id="p-username-txt"></h2>
                <button onclick="logout()" style="border:none; background:#eee; padding:8px 15px; border-radius:5px;">Выйти</button>
            </div>
            <div class="photo-grid" id="p-grid-items"></div>
        </div>

        <nav>
            <i class="fa-solid fa-house nav-link active" onclick="switchTab('home', this)"></i>
            <i class="fa-solid fa-clapperboard nav-link" onclick="switchTab('shorts', this)"></i>
            <i class="fa-solid fa-square-plus nav-link" onclick="openUploadModal()"></i>
            <i class="fa-solid fa-circle-user nav-link" onclick="switchTab('profile', this)"></i>
        </nav>
    </div>

    <div id="upload-modal" class="modal">
        <div class="modal-content">
            <h3>Создать пост</h3>
            <select id="upload-type">
                <option value="post">Фото в ленту</option>
                <option value="short">Shorts (Видео)</option>
            </select>
            <input type="file" id="upload-file" accept="image/*,video/*">
            <textarea id="upload-desc" placeholder="Описание..."></textarea>
            <button class="btn-action" id="upload-btn" onclick="processUpload()">Опубликовать</button>
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

        // AUTH FUNCTIONS
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

        // NAVIGATION
        window.switchTab = (tab, el) => {
            document.querySelectorAll('.view-content, #shorts-view').forEach(v => v.style.display = 'none');
            document.querySelectorAll('.nav-link').forEach(l => l.classList.remove('active'));
            document.getElementById(tab + '-view').style.display = tab === 'shorts' ? 'block' : 'block';
            if(tab === 'home') document.getElementById('home-view').style.display = 'block';
            el.classList.add('active');
            if(tab === 'profile') renderProfile();
        };

        window.openUploadModal = () => document.getElementById('upload-modal').style.display = 'flex';
        window.closeUploadModal = () => document.getElementById('upload-modal').style.display = 'none';

        // FIREBASE ACTIONS
        window.processUpload = async () => {
            const type = document.getElementById('upload-type').value;
            const file = document.getElementById('upload-file').files[0];
            const desc = document.getElementById('upload-desc').value;
            if(!file) return alert("Файл!");

            const r = new FileReader();
            r.onload = async e => {
                await addDoc(collection(db, type === 'post' ? "posts" : "shorts"), {
                    author: me.name, ava: me.ava, content: e.target.result, text: desc,
                    likes: 0, likedUsers: [], comments: [], time: Date.now()
                });
                closeUploadModal();
            };
            r.readAsDataURL(file);
        };

        window.toggleLike = async (id, likedUsers) => {
            const postRef = doc(db, "posts", id);
            const isLiked = likedUsers.includes(me.name);
            await updateDoc(postRef, {
                likedUsers: isLiked ? likedUsers.filter(u => u !== me.name) : arrayUnion(me.name)
            });
        };

        window.addComment = async (id) => {
            const val = document.getElementById('comm-in-'+id).value;
            if(!val) return;
            await updateDoc(doc(db, "posts", id), { comments: arrayUnion({ user: me.name, text: val }) });
        };

        // RENDER LOGIC
        if(me) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-container').style.display = 'block';

            // Feed
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
                            <strong>${p.author}</strong> ${p.text}
                            <div class="comments-section">
                                ${p.comments?.map(c => `<div class="comment"><b>${c.user}</b> ${c.text}</div>`).join('') || ''}
                                <div class="comment-input-area">
                                    <input type="text" id="comm-in-${d.id}" placeholder="Коммент...">
                                    <button onclick="addComment('${d.id}')" style="border:none; color:var(--fb-blue); background:none; font-weight:bold;">ОК</button>
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
                    <div class="short-video-container">
                        <video class="short-video" src="${v.content}" loop autoplay muted onclick="this.paused?this.play():this.pause()"></video>
                        <div class="short-info"><h3>@${v.author}</h3><p>${v.text}</p></div>
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
                    .map(d => `<img src="${d.data().content}">`).join('');
            });
        };

        window.logout = () => { localStorage.clear(); location.reload(); };
    </script>
</body>
</html>

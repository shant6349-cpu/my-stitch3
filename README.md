<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Ultimate OS</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --fb-blue: #0866ff; --inst-pink: #e1306c; --bg: #f0f2f5; --border: #dbdbdb; --white: #ffffff; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica; margin: 0; background: var(--bg); color: #1c1e21; overflow-x: hidden; }

        /* --- ЭКРАН РЕГИСТРАЦИИ --- */
        #auth-screen { position: fixed; inset: 0; background: var(--white); z-index: 9999; display: flex; align-items: center; justify-content: center; }
        .auth-card { width: 90%; max-width: 400px; padding: 40px; border-radius: 15px; box-shadow: 0 15px 35px rgba(0,0,0,0.1); text-align: center; border: 1px solid var(--border); }
        .auth-card h1 { font-style: italic; font-size: 45px; color: var(--fb-blue); margin-bottom: 10px; }
        input, textarea { width: 100%; padding: 14px; margin-bottom: 15px; border: 1px solid var(--border); border-radius: 8px; font-size: 16px; box-sizing: border-box; background: #f5f6f7; }
        .btn-action { background: var(--fb-blue); color: white; border: none; padding: 15px; border-radius: 8px; width: 100%; font-weight: bold; font-size: 18px; cursor: pointer; transition: 0.3s; }
        .btn-action:hover { opacity: 0.9; }

        /* --- ГЛАВНАЯ СТРУКТУРА --- */
        #app-container { display: none; }
        header { background: var(--white); padding: 10px 20px; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; position: sticky; top: 0; z-index: 1000; }
        .view-content { max-width: 600px; margin: 0 auto; padding-bottom: 80px; }

        /* --- STORIES --- */
        .stories-container { display: flex; gap: 15px; padding: 15px; background: var(--white); border-bottom: 1px solid var(--border); overflow-x: auto; scrollbar-width: none; }
        .story-item { width: 70px; flex-shrink: 0; text-align: center; cursor: pointer; }
        .story-circle { width: 66px; height: 66px; border-radius: 50%; border: 3px solid var(--inst-pink); padding: 2px; position: relative; }
        .story-circle img { width: 100%; height: 100%; border-radius: 50%; object-fit: cover; }
        .story-plus { position: absolute; bottom: 0; right: 0; background: var(--fb-blue); color: white; border-radius: 50%; width: 22px; height: 22px; border: 2px solid white; display: flex; align-items: center; justify-content: center; font-size: 12px; }
        .story-name { font-size: 12px; margin-top: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

        /* --- POSTS --- */
        .post { background: var(--white); border: 1px solid var(--border); border-radius: 10px; margin: 20px 0; overflow: hidden; }
        .post-header { padding: 12px; display: flex; align-items: center; gap: 10px; font-weight: 600; }
        .user-ava { width: 36px; height: 36px; border-radius: 50%; object-fit: cover; border: 1px solid #eee; }
        .post-img { width: 100%; display: block; max-height: 600px; object-fit: contain; background: #000; }
        .post-footer { padding: 15px; }
        .post-icons { display: flex; gap: 20px; font-size: 24px; margin-bottom: 10px; }
        .fa-heart.active { color: #ff3040; animation: pulse 0.3s; }
        @keyframes pulse { 0% {transform: scale(1);} 50% {transform: scale(1.3);} 100% {transform: scale(1);} }

        /* --- COMMENTS --- */
        .comments-section { border-top: 1px solid #f0f0f0; padding-top: 10px; margin-top: 10px; }
        .comment { font-size: 14px; margin-bottom: 5px; }
        .comment b { margin-right: 5px; }
        .comment-input-area { display: flex; gap: 10px; margin-top: 10px; }
        .comment-input-area input { margin-bottom: 0; padding: 8px; font-size: 14px; }

        /* --- SHORTS (TIKTOK STYLE) --- */
        #shorts-view { display: none; background: #000; height: calc(100vh - 70px); position: relative; overflow-y: scroll; scroll-snap-type: y mandatory; }
        .short-video-container { height: 100%; scroll-snap-align: start; position: relative; display: flex; align-items: center; justify-content: center; }
        .short-video { width: 100%; height: 100%; object-fit: cover; }
        .short-info { position: absolute; bottom: 20px; left: 15px; color: white; text-shadow: 1px 1px 5px rgba(0,0,0,0.8); }
        .short-side-btns { position: absolute; right: 15px; bottom: 80px; display: flex; flex-direction: column; gap: 25px; color: white; align-items: center; }

        /* --- PROFILE --- */
        #profile-view { display: none; }
        .profile-top { padding: 40px 20px; background: white; text-align: center; border-bottom: 1px solid var(--border); }
        .p-avatar { width: 120px; height: 120px; border-radius: 50%; border: 4px solid white; box-shadow: 0 5px 15px rgba(0,0,0,0.1); margin-bottom: 15px; object-fit: cover; }
        .photo-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 2px; }
        .photo-grid div { aspect-ratio: 1/1; position: relative; }
        .photo-grid img, .photo-grid video { width: 100%; height: 100%; object-fit: cover; }

        /* --- MODALS --- */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 5000; align-items: center; justify-content: center; }
        .modal-content { background: white; padding: 25px; border-radius: 15px; width: 90%; max-width: 450px; }

        /* --- NAVIGATION --- */
        nav { position: fixed; bottom: 0; width: 100%; background: white; border-top: 1px solid var(--border); display: flex; justify-content: space-around; padding: 12px 0; z-index: 1000; }
        .nav-link { font-size: 26px; color: #65676b; cursor: pointer; transition: 0.2s; }
        .nav-link.active { color: var(--fb-blue); }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <h1>StitchGram</h1>
            <p>Добро пожаловать в новую эру</p>
            <input type="text" id="reg-user" placeholder="Имя пользователя">
            <input type="password" id="reg-pass" placeholder="Пароль">
            <p style="font-size: 12px; color: #65676b;">Выберите аватарку с вашего ПК:</p>
            <input type="file" id="reg-file" accept="image/*" onchange="previewRegAva(this)">
            <img id="reg-ava-pre" src="" style="width:100px; height:100px; border-radius:50%; object-fit:cover; display:none; margin: 0 auto 15px;">
            <button class="btn-action" onclick="finishReg()">Создать аккаунт</button>
        </div>
    </div>

    <div id="app-container">
        <header>
            <div style="font-weight: 900; font-size: 24px; color: var(--fb-blue);">StitchGram</div>
            <div style="display: flex; gap: 15px;">
                <i class="fa-solid fa-heart nav-link"></i>
                <i class="fa-solid fa-paper-plane nav-link"></i>
            </div>
        </header>

        <div id="home-view" class="view-content active-view">
            <div class="stories-container">
                <div class="story-item" onclick="triggerStoryUpload()">
                    <div class="story-circle">
                        <img id="my-ava-top" src="">
                        <div class="story-plus">+</div>
                    </div>
                    <div class="story-name">Ваша история</div>
                </div>
                <input type="file" id="story-file-hidden" hidden accept="image/*" onchange="uploadStory(this)">
                <div id="others-stories-list" style="display: flex; gap: 15px;"></div>
            </div>
            <div id="feed-items"></div>
        </div>

        <div id="shorts-view">
            </div>

        <div id="profile-view">
            <div class="profile-top">
                <img id="p-avatar-img" class="p-avatar" src="">
                <h2 id="p-username-txt" style="margin: 0;"></h2>
                <p style="color: #65676b; margin-top: 5px;">Любитель StitchGram</p>
                <button onclick="logout()" style="background: #f0f2f5; border: none; padding: 8px 20px; border-radius: 5px; font-weight: bold; margin-top: 10px;">Выйти</button>
            </div>
            <div class="photo-grid" id="p-grid-items"></div>
        </div>

        <nav>
            <i class="fa-solid fa-house nav-link active" onclick="switchTab('home')"></i>
            <i class="fa-solid fa-clapperboard nav-link" onclick="switchTab('shorts')"></i>
            <i class="fa-solid fa-square-plus nav-link" onclick="openUploadModal()"></i>
            <i class="fa-solid fa-circle-user nav-link" onclick="switchTab('profile')"></i>
        </nav>
    </div>

    <div id="upload-modal" class="modal">
        <div class="modal-content">
            <h2 style="margin-top: 0;">Создать публикацию</h2>
            <select id="upload-type" style="width:100%; padding:12px; margin-bottom:15px; border-radius:8px; border:1px solid #ddd;">
                <option value="post">Обычный пост (Фото)</option>
                <option value="short">Shorts (Видео)</option>
            </select>
            <input type="file" id="upload-file" accept="image/*,video/*">
            <textarea id="upload-desc" placeholder="Напишите что-нибудь..."></textarea>
            <button class="btn-action" onclick="processUpload()">Опубликовать</button>
            <button onclick="closeUploadModal()" style="background:none; border:none; width:100%; color:gray; margin-top:10px; cursor:pointer;">Отмена</button>
        </div>
    </div>

    <script>
        // --- ДАННЫЕ ПРИЛОЖЕНИЯ ---
        let me = JSON.parse(localStorage.getItem('sg_me'));
        let posts = JSON.parse(localStorage.getItem('sg_posts')) || [];
        let shorts = JSON.parse(localStorage.getItem('sg_shorts')) || [];
        let stories = JSON.parse(localStorage.getItem('sg_stories')) || [];
        let tempAva = "";

        // --- РЕГИСТРАЦИЯ ---
        function previewRegAva(input) {
            const r = new FileReader();
            r.onload = e => {
                tempAva = e.target.result;
                document.getElementById('reg-ava-pre').src = e.target.result;
                document.getElementById('reg-ava-pre').style.display = 'block';
            }
            r.readAsDataURL(input.files[0]);
        }

        function finishReg() {
            const name = document.getElementById('reg-user').value.trim();
            const pass = document.getElementById('reg-pass').value.trim();
            if(!name || !pass) return alert("Заполни все поля!");

            me = { name, pass, ava: tempAva || 'https://via.placeholder.com/150' };
            localStorage.setItem('sg_me', JSON.stringify(me));
            location.reload();
        }

        // --- НАВИГАЦИЯ ---
        function switchTab(tab) {
            document.querySelectorAll('.view-content, #shorts-view, #profile-view').forEach(v => v.style.display = 'none');
            document.querySelectorAll('.nav-link').forEach(l => l.classList.remove('active'));
            
            if(tab === 'home') {
                document.getElementById('home-view').style.display = 'block';
                renderFeed();
            } else if(tab === 'shorts') {
                document.getElementById('shorts-view').style.display = 'block';
                renderShorts();
            } else if(tab === 'profile') {
                document.getElementById('profile-view').style.display = 'block';
                renderProfile();
            }
            event.currentTarget.classList.add('active');
        }

        // --- ЗАГРУЗКА ---
        function openUploadModal() { document.getElementById('upload-modal').style.display = 'flex'; }
        function closeUploadModal() { document.getElementById('upload-modal').style.display = 'none'; }

        function processUpload() {
            const type = document.getElementById('upload-type').value;
            const file = document.getElementById('upload-file').files[0];
            const desc = document.getElementById('upload-desc').value;

            if(!file) return alert("Выбери файл!");

            const r = new FileReader();
            r.onload = e => {
                const data = {
                    id: Date.now(),
                    author: me.name,
                    ava: me.ava,
                    content: e.target.result,
                    text: desc,
                    likes: 0,
                    liked: false,
                    comments: []
                };

                if(type === 'post') {
                    posts.unshift(data);
                    localStorage.setItem('sg_posts', JSON.stringify(posts));
                } else {
                    shorts.unshift(data);
                    localStorage.setItem('sg_shorts', JSON.stringify(shorts));
                }
                closeUploadModal();
                location.reload();
            }
            r.readAsDataURL(file);
        }

        // --- СТОРИС ---
        function triggerStoryUpload() { document.getElementById('story-file-hidden').click(); }
        function uploadStory(input) {
            const r = new FileReader();
            r.onload = e => {
                stories.unshift({ author: me.name, ava: me.ava, img: e.target.result });
                localStorage.setItem('sg_stories', JSON.stringify(stories));
                renderStories();
            }
            r.readAsDataURL(input.files[0]);
        }

        function renderStories() {
            const list = document.getElementById('others-stories-list');
            list.innerHTML = stories.map(s => `
                <div class="story-item">
                    <div class="story-circle"><img src="${s.img}"></div>
                    <div class="story-name">${s.author}</div>
                </div>
            `).join('');
        }

        // --- КОММЕНТАРИИ ---
        function addComment(postIdx) {
            const input = document.getElementById(`comm-in-${postIdx}`);
            if(!input.value.trim()) return;
            
            posts[postIdx].comments.push({ user: me.name, text: input.value });
            localStorage.setItem('sg_posts', JSON.stringify(posts));
            input.value = '';
            renderFeed();
        }

        // --- РЕНДЕРИНГ ---
        function renderFeed() {
            const container = document.getElementById('feed-items');
            container.innerHTML = posts.map((p, i) => `
                <div class="post">
                    <div class="post-header"><img src="${p.ava}" class="user-ava"> ${p.author}</div>
                    <img src="${p.content}" class="post-img" ondblclick="toggleLike(${i})">
                    <div class="post-footer">
                        <div class="post-icons">
                            <i class="fa-solid fa-heart ${p.liked ? 'active' : ''}" onclick="toggleLike(${i})"></i>
                            <i class="fa-regular fa-comment"></i>
                            <i class="fa-regular fa-paper-plane"></i>
                        </div>
                        <div><strong>${p.author}</strong> ${p.text}</div>
                        <div class="comments-section">
                            ${p.comments.map(c => `<div class="comment"><b>${c.user}</b> ${c.text}</div>`).join('')}
                            <div class="comment-input-area">
                                <input type="text" id="comm-in-${i}" placeholder="Добавьте комментарий...">
                                <button onclick="addComment(${i})" style="background:none; border:none; color:var(--fb-blue); font-weight:bold;">Ок</button>
                            </div>
                        </div>
                    </div>
                </div>
            `).join('');
            renderStories();
        }

        function toggleLike(i) {
            posts[i].liked = !posts[i].liked;
            localStorage.setItem('sg_posts', JSON.stringify(posts));
            renderFeed();
        }

        function renderShorts() {
            const container = document.getElementById('shorts-view');
            container.innerHTML = shorts.map(s => `
                <div class="short-video-container">
                    <video class="short-video" src="${s.content}" loop autoplay muted onclick="this.paused ? this.play() : this.pause()"></video>
                    <div class="short-info">
                        <h3>@${s.author}</h3>
                        <p>${s.text}</p>
                    </div>
                    <div class="short-side-btns">
                        <i class="fa-solid fa-heart" style="font-size:35px;"></i>
                        <i class="fa-solid fa-comment" style="font-size:35px;"></i>
                        <i class="fa-solid fa-share" style="font-size:35px;"></i>
                    </div>
                </div>
            `).join('');
        }

        function renderProfile() {
            document.getElementById('p-avatar-img').src = me.ava;
            document.getElementById('p-username-txt').innerText = me.name;
            const grid = document.getElementById('p-grid-items');
            const myPosts = posts.filter(p => p.author === me.name);
            const myShorts = shorts.filter(s => s.author === me.name);
            
            grid.innerHTML = [...myPosts, ...myShorts].map(item => `
                <div>
                    ${item.content.includes('video') ? `<video src="${item.content}"></video>` : `<img src="${item.content}">`}
                </div>
            `).join('');
        }

        function logout() { localStorage.clear(); location.reload(); }

        // --- СТАРТ ---
        if(me) {
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('app-container').style.display = 'block';
            document.getElementById('my-ava-top').src = me.ava;
            renderFeed();
        }
    </script>
</body>
</html>

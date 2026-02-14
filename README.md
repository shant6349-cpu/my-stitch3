<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Original</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --blue: #0095f6; --bg: #fafafa; --border: #dbdbdb; }
        body { font-family: -apple-system, sans-serif; margin: 0; background: var(--bg); padding-bottom: 70px; }

        /* ЭКРАН РЕГИСТРАЦИИ */
        #auth-screen { position: fixed; inset: 0; background: white; z-index: 5000; display: flex; align-items: center; justify-content: center; }
        .auth-card { width: 90%; max-width: 350px; text-align: center; padding: 25px; border: 1px solid var(--border); border-radius: 10px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        
        /* ГЛАВНЫЕ СТИЛИ */
        .screen { display: none; max-width: 500px; margin: 0 auto; }
        .active { display: block; }
        header { background: #fff; padding: 15px; border-bottom: 1px solid var(--border); text-align: center; font-weight: bold; font-size: 22px; color: var(--blue); position: sticky; top: 0; z-index: 100; }

        /* СТОРИС */
        .stories { display: flex; gap: 12px; padding: 15px; background: #fff; border-bottom: 1px solid var(--border); overflow-x: auto; }
        .s-circle { width: 62px; height: 62px; border-radius: 50%; border: 3px solid #e1306c; flex-shrink: 0; padding: 2px; cursor: pointer; position: relative; }
        .s-circle img { width: 100%; height: 100%; border-radius: 50%; object-fit: cover; }
        .plus { position: absolute; bottom: 0; right: 0; background: var(--blue); color: white; border-radius: 50%; width: 20px; height: 20px; display: flex; align-items: center; justify-content: center; border: 2px solid white; font-size: 14px; }

        /* ПОСТЫ */
        .post { background: #fff; border: 1px solid var(--border); margin: 15px 0; border-radius: 8px; }
        .post-header { padding: 10px; display: flex; align-items: center; gap: 10px; font-weight: bold; }
        .post-img { width: 100%; display: block; }
        .post-actions { padding: 12px; display: flex; gap: 15px; font-size: 24px; }
        .fa-heart.liked { color: #ed4956; }

        /* НАВИГАЦИЯ */
        nav { position: fixed; bottom: 0; width: 100%; background: #fff; border-top: 1px solid var(--border); display: flex; justify-content: space-around; padding: 15px 0; z-index: 1000; }
        .nav-btn { font-size: 26px; color: #262626; cursor: pointer; }

        input, textarea, button { width: 100%; padding: 12px; margin-top: 10px; border-radius: 8px; border: 1px solid var(--border); box-sizing: border-box; }
        button { background: var(--blue); color: white; border: none; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="auth-card">
            <h1 style="font-style: italic; color: var(--blue);">StitchGram</h1>
            <input type="text" id="reg-name" placeholder="Твой ник">
            <input type="password" id="reg-pass" placeholder="Пароль">
            <p style="font-size: 12px; color: gray;">Загрузи аватарку:</p>
            <input type="file" id="reg-ava" accept="image/*">
            <button onclick="startApp()">Создать аккаунт</button>
        </div>
    </div>

    <div id="home-screen" class="screen">
        <header>StitchGram</header>
        <div class="stories">
            <div class="s-circle" onclick="document.getElementById('st-input').click()">
                <img id="my-ava-st" src="">
                <div class="plus">+</div>
            </div>
            <input type="file" id="st-input" hidden onchange="addStory(this)">
            <div id="other-stories" style="display:flex; gap:12px;"></div>
        </div>
        <div id="feed"></div>
    </div>

    <div id="profile-screen" class="screen">
        <div style="text-align: center; padding: 30px; background: white; border-bottom: 1px solid var(--border);">
            <img id="p-ava" style="width:100px; height:100px; border-radius:50%; object-fit: cover; border: 2px solid var(--blue);">
            <h2 id="p-name"></h2>
            <button onclick="location.reload()" style="background:#eee; color:black; width:auto; padding:5px 15px;">Выход</button>
        </div>
        <div id="my-grid" style="display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 2px;"></div>
    </div>

    <div id="up-modal" style="display:none; position:fixed; inset:0; background:rgba(0,0,0,0.8); z-index:2000; align-items:center; justify-content:center;">
        <div style="background:white; padding:20px; border-radius:15px; width:300px;">
            <h3 style="margin:0 0 10px 0;">Новый пост</h3>
            <input type="file" id="up-file" accept="image/*">
            <textarea id="up-text" placeholder="Описание..."></textarea>
            <button onclick="savePost()">Опубликовать</button>
            <button onclick="document.getElementById('up-modal').style.display='none'" style="background:none; color:gray;">Отмена</button>
        </div>
    </div>

    <nav id="navbar" style="display:none">
        <i class="fa-solid fa-house nav-btn" onclick="show('home-screen')"></i>
        <i class="fa-solid fa-plus-square nav-btn" onclick="document.getElementById('up-modal').style.display='flex'"></i>
        <i class="fa-solid fa-user nav-btn" onclick="show('profile-screen')"></i>
    </nav>

    <script>
        let user = null;
        let posts = [];
        let stories = [];

        function startApp() {
            const name = document.getElementById('reg-name').value;
            const file = document.getElementById('reg-ava').files[0];
            if(!name) return alert("Введите имя!");

            const reader = new FileReader();
            reader.onload = (e) => {
                user = { name, ava: e.target.result };
                document.getElementById('auth-screen').style.display = 'none';
                document.getElementById('navbar').style.display = 'flex';
                document.getElementById('my-ava-st').src = user.ava;
                show('home-screen');
            };
            if(file) reader.readAsDataURL(file); 
            else { user = {name, ava: 'https://via.placeholder.com/100'}; 
                   document.getElementById('auth-screen').style.display = 'none';
                   document.getElementById('navbar').style.display = 'flex';
                   document.getElementById('my-ava-st').src = user.ava;
                   show('home-screen'); }
        }

        function show(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            if(id === 'home-screen') renderFeed();
            if(id === 'profile-screen') renderProfile();
        }

        function savePost() {
            const file = document.getElementById('up-file').files[0];
            const text = document.getElementById('up-text').value;
            if(!file) return alert("Выбери фото!");

            const reader = new FileReader();
            reader.onload = (e) => {
                posts.unshift({ author: user.name, ava: user.ava, img: e.target.result, text, liked: false });
                document.getElementById('up-modal').style.display = 'none';
                show('home-screen');
            };
            reader.readAsDataURL(file);
        }

        function addStory(input) {
            const reader = new FileReader();
            reader.onload = (e) => {
                stories.unshift({ src: e.target.result });
                const box = document.getElementById('other-stories');
                box.innerHTML = stories.map(s => `<div class="s-circle"><img src="${s.src}"></div>`).join('');
            };
            reader.readAsDataURL(input.files[0]);
        }

        function toggleLike(i) {
            posts[i].liked = !posts[i].liked;
            renderFeed();
        }

        function renderFeed() {
            const feed = document.getElementById('feed');
            feed.innerHTML = posts.map((p, i) => `
                <div class="post">
                    <div class="post-header"><img src="${p.ava}" style="width:30px; height:30px; border-radius:50%;"> ${p.author}</div>
                    <img src="${p.img}" class="post-img" ondblclick="toggleLike(${i})">
                    <div class="post-actions">
                        <i class="fa-solid fa-heart ${p.liked ? 'liked' : ''}" onclick="toggleLike(${i})"></i>
                        <i class="fa-regular fa-comment"></i>
                    </div>
                    <div style="padding:0 12px 12px;"><b>${p.author}</b> ${p.text}</div>
                </div>
            `).join('');
        }

        function renderProfile() {
            document.getElementById('p-ava').src = user.ava;
            document.getElementById('p-name').innerText = user.name;
            const my = posts.filter(p => p.author === user.name);
            document.getElementById('my-grid').innerHTML = my.map(m => `<img src="${m.img}" style="width:100%; aspect-ratio:1/1; object-fit:cover;">`).join('');
        }
    </script>
</body>
</html>

<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Sync Edition</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root { --blue: #0095f6; --bg: #fafafa; --border: #dbdbdb; }
        body { font-family: -apple-system, sans-serif; margin: 0; background: var(--bg); padding-bottom: 70px; }
        
        /* ЛЕНТА */
        .feed-container { max-width: 500px; margin: 0 auto; padding-top: 10px; }
        .post { background: #fff; border: 1px solid var(--border); margin-bottom: 20px; border-radius: 8px; overflow: hidden; }
        .post-header { padding: 12px; display: flex; align-items: center; gap: 10px; font-weight: bold; }
        .post-header img { width: 32px; height: 32px; border-radius: 50%; object-fit: cover; }
        .post-img { width: 100%; display: block; background: #eee; }
        .post-actions { padding: 12px; display: flex; gap: 15px; font-size: 22px; }
        .post-info { padding: 0 12px 12px; }
        .fa-heart.active { color: #ed4956; }

        /* НАВИГАЦИЯ */
        nav { position: fixed; bottom: 0; width: 100%; background: #fff; border-top: 1px solid var(--border); display: flex; justify-content: space-around; padding: 15px 0; z-index: 1000; }
        .nav-btn { font-size: 24px; color: #262626; cursor: pointer; }
    </style>
</head>
<body>

    <header style="background:#fff; border-bottom:1px solid #dbdbdb; padding:15px; text-align:center; position:sticky; top:0; z-index:100;">
        <b style="color:var(--blue); font-size:20px;">StitchGram</b>
    </header>

    <div class="feed-container" id="main-feed">
        </div>

    <nav>
        <i class="fa-solid fa-house nav-btn"></i>
        <i class="fa-solid fa-square-plus nav-btn" onclick="openUpload()"></i>
        <i class="fa-solid fa-circle-user nav-btn"></i>
    </nav>

    <div id="upload-modal" style="display:none; position:fixed; inset:0; background:rgba(0,0,0,0.8); z-index:2000; align-items:center; justify-content:center;">
        <div style="background:#white; padding:20px; border-radius:15px; width:300px; background:white;">
            <h3>Новый пост</h3>
            <input type="file" id="post-file" accept="image/*">
            <textarea id="post-text" placeholder="Описание..." style="width:100%; margin:10px 0; padding:10px; box-sizing:border-box;"></textarea>
            <button onclick="publish()" style="width:100%; background:var(--blue); color:white; border:none; padding:10px; border-radius:5px; font-weight:bold;">Опубликовать</button>
            <button onclick="closeUpload()" style="width:100%; background:none; color:gray; border:none; margin-top:10px;">Отмена</button>
        </div>
    </div>

    <script>
        // Имитация базы данных (пока без Firebase)
        let localPosts = [];

        function openUpload() { document.getElementById('upload-modal').style.display = 'flex'; }
        function closeUpload() { document.getElementById('upload-modal').style.display = 'none'; }

        function publish() {
            const file = document.getElementById('post-file').files[0];
            const text = document.getElementById('post-text').value;
            if(!file) return alert("Выбери фото!");

            const reader = new FileReader();
            reader.onload = (e) => {
                const newPost = {
                    id: Date.now(),
                    img: e.target.result,
                    desc: text,
                    liked: false
                };
                localPosts.unshift(newPost);
                renderFeed();
                closeUpload();
            };
            reader.readAsDataURL(file);
        }

        function toggleLike(id) {
            const post = localPosts.find(p => p.id === id);
            if(post) post.liked = !post.liked;
            renderFeed();
        }

        function renderFeed() {
            const feed = document.getElementById('main-feed');
            feed.innerHTML = localPosts.map(p => `
                <div class="post">
                    <div class="post-header"><img src="https://via.placeholder.com/32"> <span>User</span></div>
                    <img src="${p.img}" class="post-img">
                    <div class="post-actions">
                        <i class="fa-${p.liked ? 'solid' : 'regular'} fa-heart ${p.liked ? 'active' : ''}" onclick="toggleLike(${p.id})"></i>
                        <i class="fa-regular fa-comment"></i>
                    </div>
                    <div class="post-info"><b>User</b> ${p.desc}</div>
                </div>
            `).join('');
        }
    </script>
</body>
</html>

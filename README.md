<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Pro</title>
    <style>
        :root {
            --bg: #fafafa;
            --blue: #0095f6;
            --border: #dbdbdb;
        }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; margin: 0; background: var(--bg); color: #262626; padding-bottom: 70px; }
        
        /* Шапка */
        header { background: white; padding: 12px 0; text-align: center; border-bottom: 1px solid var(--border); font-weight: bold; font-size: 20px; color: var(--blue); position: sticky; top: 0; z-index: 100; letter-spacing: 1px; }

        /* Лента */
        .feed { max-width: 500px; margin: 10px auto; padding: 0 10px; }
        .post { background: white; border: 1px solid var(--border); margin-bottom: 15px; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 5px rgba(0,0,0,0.05); animation: fadeIn 0.5s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        .post img { width: 100%; height: auto; display: block; background: #eee; }
        .post-info { padding: 12px; }
        .post-info b { color: #262626; margin-right: 5px; }
        .post-info p { margin: 5px 0 0; color: #555; font-size: 14px; line-height: 1.4; }

        /* Нижнее меню */
        .nav-bar { position: fixed; bottom: 0; width: 100%; background: white; border-top: 1px solid var(--border); display: flex; justify-content: space-around; padding: 10px 0; z-index: 1000; }
        .nav-item { border: none; background: none; font-size: 24px; cursor: pointer; color: #262626; transition: 0.2s; }
        .nav-item:active { transform: scale(0.8); color: var(--blue); }

        /* Окно добавления */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 2000; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
        .modal-content { background: white; padding: 25px; border-radius: 20px; width: 85%; max-width: 350px; text-align: center; }
        .modal-content h3 { margin-top: 0; color: var(--blue); }
        .modal-content input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid var(--border); border-radius: 8px; box-sizing: border-box; outline: none; }
        .modal-content input:focus { border-color: var(--blue); }
        .btn-send { width: 100%; background: var(--blue); color: white; border: none; padding: 12px; border-radius: 8px; font-weight: bold; margin-top: 10px; cursor: pointer; }
        .btn-cancel { background: none; border: none; color: #ed4956; margin-top: 10px; font-size: 14px; cursor: pointer; }
    </style>
</head>
<body>

    <header>STITCHGRAM</header>

    <div class="feed" id="feed">
        <p style="text-align: center; color: gray; margin-top: 50px;">Загрузка контента...</p>
    </div>

    <div class="nav-bar">
        <button class="nav-item" onclick="location.reload()">🏠</button>
        <button class="nav-item" onclick="document.getElementById('modal').style.display='flex'" style="font-size: 30px; font-weight: bold; color: #0095f6;">➕</button>
        <button class="nav-item" onclick="alert('Профиль скоро будет!')">👤</button>
    </div>

    <div id="modal" class="modal">
        <div class="modal-content">
            <h3>Создать пост</h3>
            <input type="text" id="url" placeholder="Ссылка на фото (URL)">
            <input type="text" id="desc" placeholder="Описание к фото">
            <button id="publish" class="btn-send">Опубликовать</button>
            <button onclick="document.getElementById('modal').style.display='none'" class="btn-cancel">Отмена</button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, query, orderBy, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        // ТВОИ КЛЮЧИ
        const firebaseConfig = {
            apiKey: "AIzaSyAHxAc0XUrq4N5nuH0UnD66PAnUXRU_DJQ",
            authDomain: "myprogram-1b6b0.firebaseapp.com",
            projectId: "myprogram-1b6b0",
            appId: "1:678957156444:web:6ac76957ea7e0ba2283e99"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        // Публикация
        document.getElementById('publish').onclick = async () => {
            const urlInput = document.getElementById('url');
            const descInput = document.getElementById('desc');
            
            if(!urlInput.value || !descInput.value) return alert("Заполни все поля!");

            const btn = document.getElementById('publish');
            btn.innerText = "Публикация...";
            btn.disabled = true;

            try {
                await addDoc(collection(db, "posts"), {
                    url: urlInput.value,
                    text: descInput.value,
                    time: Date.now()
                });
                document.getElementById('modal').style.display = 'none';
                urlInput.value = "";
                descInput.value = "";
            } catch (e) { alert("Ошибка! Проверь Rules в Firebase"); }
            
            btn.innerText = "Опубликовать";
            btn.disabled = false;
        };

        // Подгрузка постов
        const q = query(collection(db, "posts"), orderBy("time", "desc"));
        onSnapshot(q, (snapshot) => {
            const feed = document.getElementById('feed');
            feed.innerHTML = snapshot.empty ? '<p style="text-align:center; padding-top:50px">Лента пуста. Добавь первый пост!</p>' : "";
            snapshot.forEach(doc => {
                const data = doc.data();
                feed.innerHTML += `
                    <div class="post">
                        <img src="${data.url}" onerror="this.src='https://placehold.co/400x400?text=Ошибка+загрузки+фото'">
                        <div class="post-info">
                            <b>Пользователь:</b>
                            <p>${data.text}</p>
                        </div>
                    </div>`;
            });
        });
    </script>
</body>
</html>

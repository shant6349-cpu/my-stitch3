<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Premium</title>
    <style>
        :root { 
            --blue: #0095f6; 
            --border: rgba(219, 219, 219, 0.5);
            /* Красивый градиент для фона */
            --gradient: linear-gradient(135deg, #e0eafc 0%, #cfdef3 100%);
        }

        body { 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; 
            margin: 0; 
            background: var(--gradient); 
            background-attachment: fixed; /* Чтобы фон не дергался при скролле */
            padding-bottom: 80px; 
            color: #262626;
        }

        header { 
            background: rgba(255, 255, 255, 0.9); 
            backdrop-filter: blur(10px); /* Эффект размытия */
            padding: 15px; 
            text-align: center; 
            border-bottom: 1px solid var(--border); 
            font-weight: bold; 
            color: var(--blue); 
            position: sticky; 
            top: 0; 
            z-index: 10; 
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        }
        
        .feed { max-width: 500px; margin: 0 auto; padding: 10px; }

        /* Стильные карточки постов */
        .post { 
            background: rgba(255, 255, 255, 0.95); 
            border: 1px solid var(--border); 
            border-radius: 16px; 
            margin-bottom: 25px; 
            overflow: hidden; 
            box-shadow: 0 10px 20px rgba(0,0,0,0.08); /* Мягкая тень */
        }

        .post img { width: 100%; display: block; border-bottom: 1px solid #f0f0f0; }
        
        .post-content { padding: 15px; }
        
        .actions { display: flex; gap: 20px; margin-bottom: 12px; font-size: 22px; }
        
        .like-btn { cursor: pointer; border: none; background: none; padding: 0; filter: drop-shadow(0 2px 2px rgba(0,0,0,0.1)); }
        
        .comment-sec { 
            background: rgba(0,0,0,0.02); 
            border-radius: 10px; 
            padding: 10px; 
            margin-top: 12px; 
            font-size: 13px; 
        }

        .comm-input { 
            width: 100%; 
            border: none; 
            padding: 10px 0; 
            outline: none; 
            background: transparent;
            border-bottom: 1px solid var(--border); 
            margin-top: 5px;
        }

        .nav-bar { 
            position: fixed; 
            bottom: 0; 
            width: 100%; 
            background: rgba(255, 255, 255, 0.95); 
            backdrop-filter: blur(10px);
            border-top: 1px solid var(--border); 
            display: flex; 
            justify-content: space-around; 
            padding: 15px 0; 
            box-shadow: 0 -2px 10px rgba(0,0,0,0.05);
        }

        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); align-items: center; justify-content: center; z-index: 100; backdrop-filter: blur(5px); }
        .modal-box { background: white; padding: 25px; border-radius: 20px; width: 320px; text-align: center; box-shadow: 0 20px 40px rgba(0,0,0,0.3); }
        
        input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid var(--border); border-radius: 10px; box-sizing: border-box; background: #f9f9f9; }
        
        .publish-btn { width: 100%; background: var(--blue); color: white; border: none; padding: 12px; border-radius: 10px; font-weight: bold; cursor: pointer; margin-top: 10px; }
    </style>
</head>
<body>

    <header id="user-display">STITCHGRAM</header>
    
    <div class="feed" id="feed">Загрузка стильной ленты...</div>

    <div class="nav-bar">
        <button onclick="window.scrollTo({top: 0, behavior: 'smooth'})" style="border:none; background:none; font-size:24px; cursor:pointer;">🏠</button>
        <button onclick="document.getElementById('m').style.display='flex'" style="border:none; background:none; font-size:30px; color:var(--blue); cursor:pointer;">➕</button>
    </div>

    <div id="m" class="modal">
        <div class="modal-box">
            <h3>Поделиться фото</h3>
            <input type="text" id="p-url" placeholder="Вставь ссылку на картинку">
            <input type="text" id="p-desc" placeholder="Напиши что-нибудь...">
            <button class="publish-btn" onclick="sendPost()">Опубликовать</button>
            <button onclick="document.getElementById('m').style.display='none'" style="border:none; background:none; color:gray; margin-top:15px; cursor:pointer;">Закрыть</button>
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

        let userName = localStorage.getItem('stitchName');
        if(!userName) {
            userName = prompt("Как тебя называть в StitchGram?") || "Друг";
            localStorage.setItem('stitchName', userName);
        }
        document.getElementById('user-display').innerText = `Лента для ${userName}`;

        window.sendPost = async () => {
            const url = document.getElementById('p-url').value;
            const desc = document.getElementById('p-desc').value;
            if(!url || !desc) return alert("Заполни все поля!");
            
            await addDoc(collection(db, "posts"), {
                url, desc, user: userName, time: Date.now(), likes: 0, comments: []
            });
            document.getElementById('m').style.display='none';
            document.getElementById('p-url').value = "";
            document.getElementById('p-desc').value = "";
        };

        window.doLike = async (id, count) => {
            await updateDoc(doc(db, "posts", id), { likes: count + 1 });
        };

        window.addComm = async (e, id) => {
            if(e.key === 'Enter' && e.target.value) {
                await updateDoc(doc(db, "posts", id), {
                    comments: arrayUnion({ user: userName, text: e.target.value })
                });
                e.target.value = "";
            }
        };

        const q = query(collection(db, "posts"), orderBy("time", "desc"));
        onSnapshot(q, (s) => {
            const feed = document.getElementById('feed');
            feed.innerHTML = "";
            s.forEach(d => {
                const p = d.data();
                const id = d.id;
                let commsHtml = p.comments?.map(c => `<div style="margin-bottom:5px"><b>${c.user}</b> ${c.text}</div>`).join('') || "";
                
                feed.innerHTML += `
                    <div class="post">
                        <img src="${p.url}" onerror="this.src='https://via.placeholder.com/400?text=Ошибка+ссылки'">
                        <div class="post-content">
                            <div class="actions">
                                <button class="like-btn" onclick="doLike('${id}', ${p.likes})">❤️ ${p.likes}</button>
                                <span>💬 ${p.comments?.length || 0}</span>
                            </div>
                            <div style="margin-bottom:10px;"><b>${p.user}</b> ${p.desc}</div>
                            <div class="comment-sec">
                                ${commsHtml}
                                <input class="comm-input" placeholder="Добавить коммент..." onkeypress="addComm(event, '${id}')">
                            </div>
                        </div>
                    </div>`;
            });
        });
    </script>
</body>
</html>

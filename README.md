<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>StitchGram Free</title>
    <style>
        body { font-family: sans-serif; background: #fafafa; margin: 0; padding-bottom: 70px; }
        header { background: white; padding: 15px; text-align: center; border-bottom: 1px solid #dbdbdb; color: #0095f6; font-weight: bold; position: sticky; top: 0; }
        .feed { max-width: 450px; margin: 20px auto; padding: 10px; }
        .post { background: white; border: 1px solid #dbdbdb; margin-bottom: 20px; border-radius: 8px; padding: 10px; }
        .post img { width: 100%; border-radius: 5px; }
        .add-btn { position: fixed; bottom: 20px; right: 20px; background: #0095f6; color: white; width: 60px; height: 60px; border-radius: 50%; border: none; font-size: 30px; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
    </style>
</head>
<body>
    <header>STITCHGRAM (CLOUD DATA)</header>
    <div class="feed" id="feed">Загрузка ленты...</div>
    <button class="add-btn" onclick="addNewPost()">+</button>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, query, orderBy, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        // ТВОИ КЛЮЧИ ИЗ ОБЛАКА
        const firebaseConfig = {
            apiKey: "AIzaSyAHxAc0XUrq4N5nuH0UnD66PAnUXRU_DJQ",
            authDomain: "myprogram-1b6b0.firebaseapp.com",
            projectId: "myprogram-1b6b0",
            appId: "1:678957156444:web:6ac76957ea7e0ba2283e99"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        // Функция добавления через ссылку
        window.addNewPost = async () => {
            const url = prompt("Вставь ссылку на картинку из интернета:");
            const text = prompt("Напиши описание:");
            if(url && text) {
                await addDoc(collection(db, "posts"), { url, text, time: Date.now() });
            }
        };

        // Живое обновление у всех пользователей
        const q = query(collection(db, "posts"), orderBy("time", "desc"));
        onSnapshot(q, (s) => {
            const f = document.getElementById('feed');
            f.innerHTML = s.empty ? "Пока постов нет. Будь первым!" : "";
            s.forEach(d => {
                const p = d.data();
                f.innerHTML += `<div class="post"><img src="${p.url}"><p><b>Описание:</b> ${p.text}</p></div>`;
            });
        });
    </script>
</body>
</html>

<script type="module">
    // Подключаем Firebase
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { getFirestore, collection, addDoc, query, orderBy, onSnapshot, updateDoc, doc } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

    // Регистрация (оставляем локальной для простоты)
    window.previewRegAva = (input) => {
        const r = new FileReader();
        r.onload = e => {
            tempAva = e.target.result;
            document.getElementById('reg-ava-pre').src = e.target.result;
            document.getElementById('reg-ava-pre').style.display = 'block';
        }
        r.readAsDataURL(input.files[0]);
    };

    window.finishReg = () => {
        const name = document.getElementById('reg-user').value.trim();
        if(!name) return alert("Введи имя!");
        me = { name, ava: tempAva || 'https://via.placeholder.com/150' };
        localStorage.setItem('sg_me', JSON.stringify(me));
        location.reload();
    };

    // Навигация
    window.switchTab = (tab) => {
        document.querySelectorAll('.view-content, #shorts-view, #profile-view').forEach(v => v.style.display = 'none');
        document.querySelectorAll('nav i').forEach(l => l.classList.remove('active'));
        document.getElementById(tab + '-view').style.display = 'block';
        event.currentTarget.classList.add('active');
        if(tab === 'profile') renderProfile();
    };

    window.openUploadModal = () => document.getElementById('upload-modal').style.display = 'flex';
    window.closeUploadModal = () => document.getElementById('upload-modal').style.display = 'none';

    // ЗАГРУЗКА В ОБЛАКО
    window.processUpload = async () => {
        const type = document.getElementById('upload-type').value;
        const file = document.getElementById('upload-file').files[0];
        const desc = document.getElementById('upload-desc').value;
        if(!file) return alert("Выбери файл!");

        const r = new FileReader();
        r.onload = async e => {
            await addDoc(collection(db, type === 'post' ? "posts" : "shorts"), {
                author: me.name,
                ava: me.ava,
                content: e.target.result,
                text: desc,
                likes: 0,
                liked: false,
                comments: [],
                time: Date.now()
            });
            closeUploadModal();
        };
        r.readAsDataURL(file);
    };

    // ЛАЙК И КОММЕНТ
    window.toggleLike = async (id, currentLiked, currentLikes) => {
        await updateDoc(doc(db, "posts", id), { 
            liked: !currentLiked,
            likes: currentLiked ? currentLikes - 1 : currentLikes + 1 
        });
    };

    window.addComment = async (id) => {
        const input = document.getElementById(`comm-in-${id}`);
        if(!input.value.trim()) return;
        // Тут логика для массива комментов в Firebase
        alert("Комментарии почти готовы!");
    };

    // ОТОБРАЖЕНИЕ (ОБЛАЧНОЕ)
    if(me) {
        document.getElementById('auth-screen').style.display = 'none';
        document.getElementById('app-container').style.display = 'block';
        document.getElementById('my-ava-top').src = me.ava;

        // Слушаем посты
        const qPosts = query(collection(db, "posts"), orderBy("time", "desc"));
        onSnapshot(qPosts, (s) => {
            const container = document.getElementById('feed-items');
            container.innerHTML = s.docs.map(d => {
                const p = d.data();
                return `
                <div class="post">
                    <div class="post-header"><img src="${p.ava}" class="user-ava"> ${p.author}</div>
                    <img src="${p.content}" class="post-img" ondblclick="toggleLike('${d.id}', ${p.liked}, ${p.likes})">
                    <div class="post-footer">
                        <div class="post-icons">
                            <i class="fa-solid fa-heart ${p.liked ? 'active' : ''}" onclick="toggleLike('${d.id}', ${p.liked}, ${p.likes})"></i>
                            <i class="fa-regular fa-comment"></i>
                        </div>
                        <div><strong>${p.author}</strong> ${p.text}</div>
                    </div>
                </div>`;
            }).join('');
        });

        // Слушаем Shorts
        const qShorts = query(collection(db, "shorts"), orderBy("time", "desc"));
        onSnapshot(qShorts, (s) => {
            const container = document.getElementById('shorts-view');
            container.innerHTML = s.docs.map(d => {
                const v = d.data();
                return `
                <div class="short-video-container">
                    <video class="short-video" src="${v.content}" loop autoplay muted></video>
                    <div class="short-info"><h3>@${v.author}</h3><p>${v.text}</p></div>
                </div>`;
            }).join('');
        });
    }

    window.logout = () => { localStorage.clear(); location.reload(); };
</script>

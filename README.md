<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StitchGram Kawaii Edition</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* --- МИЛАЯ ЦВЕТОВАЯ ПАЛИТРА --- */
        :root { 
            --main-pink: #ffcce5; 
            --soft-purple: #e2d1f9;
            --soft-blue: #d1e9ff;
            --accent: #ff85a1;
            --bg: #fffafa; /* Цвет "белый лепесток" */
            --white: #ffffff;
            --text: #5a4a5a;
        }

        body { 
            font-family: 'Comic Sans MS', 'Segoe UI', cursive, sans-serif; 
            margin: 0; 
            background: linear-gradient(135deg, var(--main-pink), var(--soft-purple), var(--soft-blue));
            background-attachment: fixed;
            color: var(--text);
            overflow-x: hidden;
        }

        /* --- АВТОРИЗАЦИЯ (МИЛАЯ) --- */
        #auth-screen { 
            position: fixed; inset: 0; background: var(--bg); z-index: 9999; 
            display: flex; align-items: center; justify-content: center; 
        }
        .auth-card { 
            width: 85%; max-width: 380px; padding: 40px; 
            background: var(--white); border-radius: 30px; 
            box-shadow: 0 10px 30px rgba(255, 133, 161, 0.2); 
            text-align: center; border: 4px solid var(--main-pink);
        }
        .auth-card h1 { color: var(--accent); font-size: 38px; margin-bottom: 20px; text-shadow: 2px 2px #fff; }

        /* --- ИНПУТЫ И КНОПКИ --- */
        input, textarea, select { 
            width: 100%; padding: 12px; margin-bottom: 15px; 
            border: 2px solid var(--soft-purple); border-radius: 20px; 
            background: #fff; font-size: 16px; outline: none; transition: 0.3s;
        }
        input:focus { border-color: var(--accent); box-shadow: 0 0 10px var(--main-pink); }

        .btn-action { 
            background: var(--accent); color: white; border: none; 
            padding: 15px; border-radius: 25px; width: 100%; 
            font-weight: bold; font-size: 18px; cursor: pointer; 
            box-shadow: 0 4px 15px rgba(255, 133, 161, 0.4); transition: 0.3s;
        }
        .btn-action:hover { transform: scale(1.05); background: #ff6b8d; }

        /* --- ШАПКА --- */
        header { 
            background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px);
            padding: 15px 20px; border-bottom: 3px solid var(--main-pink); 
            display: flex; justify-content: space-between; align-items: center; 
            position: sticky; top: 0; z-index: 1000;
        }
        header div { font-size: 26px; color: var(--accent); letter-spacing: 2px; }

        /* --- ПОСТЫ (КАРТОЧКИ) --- */
        .view-content { max-width: 500px; margin: 0 auto; padding: 15px; padding-bottom: 80px; }
        
        .post { 
            background: var(--white); border-radius: 30px; 
            margin-bottom: 25px; overflow: hidden; 
            box-shadow: 0 8px 20px rgba(0,0,0,0.05);
            border: 2px solid #fff; animation: popIn 0.5s ease;
        }
        @keyframes popIn {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }

        .post-header { padding: 15px; display: flex; align-items: center; gap: 12px; font-weight: bold; }
        .user-ava { width: 45px; height: 45px; border-radius: 50%; border: 2px solid var(--main-pink); }
        
        .post-img { width: 100%; display: block; border-radius: 0 0 20px 20px; }

        .post-footer { padding: 20px; }
        .post-icons { display: flex; gap: 20px; font-size: 28px; margin-bottom: 10px; color: var(--accent); }
        
        .fa-heart.active { color: #ff3040; text-shadow: 0 0 10px rgba(255,48,64,0.4); }

        /* --- НАВИГАЦИЯ (НИЖНЯЯ) --- */
        nav { 
            position: fixed; bottom: 15px; left: 15px; right: 15px;
            background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(15px);
            border-radius: 40px; display: flex; justify-content: space-around; 
            padding: 12px 0; z-index: 1000; box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            border: 2px solid var(--main-pink);
        }
        .nav-link { font-size: 26px; color: #d1b2d1; cursor: pointer; transition: 0.3s; }
        .nav-link.active { color: var(--accent); transform: translateY(-5px); }

        /* --- ПРОФИЛЬ --- */
        .profile-top { padding: 30px 20px; text-align: center; }
        .p-avatar { 
            width: 110px; height: 110px; border-radius: 50%; 
            border: 5px solid white; box-shadow: 0 5px 15px var(--main-pink);
            margin-bottom: 15px;
        }
        .photo-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; padding: 10px; }
        .photo-grid img { width: 100%; aspect-ratio: 1/1; object-fit: cover; border-radius: 15px; border: 2px solid white; }

        /* --- КОММЕНТАРИИ --- */
        .comments-section { 
            background: var(--bg); border-radius: 20px; padding: 10px; margin-top: 15px; font-size: 14px; 
        }
        .comment-input-area button { color: var(--accent) !important; }

        /* --- СКРОЛЛБАР --- */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: var(--main-pink); border-radius: 10px; }
    </style>
</head>

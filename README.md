<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Stitch: Magic Edition</title>
    <style>
        /* АНИМИРОВАННЫЙ ФОН */
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            /* Цвета перелива */
            background: linear-gradient(-45deg, #0f0c29, #302b63, #24243e, #1a2a6c, #b21f1f);
            background-size: 400% 400%;
            animation: gradientBG 15s ease infinite;
            font-family: 'Segoe UI', sans-serif;
            overflow: hidden;
        }

        @keyframes gradientBG {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        #pet-stage {
            width: 400px;
            padding: 25px;
            background: rgba(255, 255, 255, 0.1); /* Полупрозрачность */
            backdrop-filter: blur(20px); /* Эффект матового стекла */
            border-radius: 40px;
            text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.2);
            box-shadow: 0 25px 80px rgba(0,0,0,0.5);
            color: white;
        }

        /* ИНДИКАТОРЫ */
        .stats-grid {
            display: grid; 
            grid-template-columns: 1fr 1fr; 
            gap: 15px; 
            margin-bottom: 20px;
        }
        .stat-box { text-align: left; }
        .stat-label { font-size: 10px; font-weight: bold; color: rgba(255,255,255,0.7); margin-bottom: 4px; display: block; }
        .bar-bg { width: 100%; height: 8px; background: rgba(0,0,0,0.3); border-radius: 5px; overflow: hidden; }
        .fill { height: 100%; width: 100%; transition: width 0.5s ease; box-shadow: 0 0 10px rgba(255,255,255,0.2); }

        /* ЛИЦО СТИЧА */
        #stitch-face {
            position: relative; 
            width: 200px; 
            height: 180px; 
            margin: 50px auto 30px;
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .head {
            position: absolute; width: 100%; height: 100%;
            background: radial-gradient(circle at 35% 30%, #5d8aff, #3a56d4 60%, #1e2a78 100%);
            border-radius: 50% 50% 46% 46%; z-index: 5;
            box-shadow: inset -5px -10px 20px rgba(0,0,0,0.6), 0 10px 20px rgba(0,0,0,0.3);
        }

        .ear {
            position: absolute; top: -20px; width: 60px; height: 130px;
            background: #3a56d4; border-radius: 100% 20% 100% 20%; z-index: 1;
            transition: transform 0.6s ease;
        }
        .ear::after {
            content: ''; position: absolute; top: 15%; left: 15%; width: 70%; height: 75%;
            background: radial-gradient(ellipse, #ff9bb7 10%, #3a56d4 80%);
            border-radius: inherit; opacity: 0.5;
        }
        .ear.left { left: -40px; transform: rotate(-35deg); }
        .ear.right { right: -40px; transform: rotate(35deg) scaleX(-1); }

        .eye-patch {
            position: absolute; top: 30px; width: 70px; height: 90px;
            background: rgba(173, 216, 230, 0.2); border-radius: 50%; z-index: 6;
        }
        .eye-patch.left { left: 18px; transform: rotate(12deg); }
        .eye-patch.right { right: 18px; transform: rotate(-12deg); }

        .eye {
            position: absolute; top: 18px; left: 12px; width: 46px; height: 58px;
            background: #080808; border-radius: 50%; transition: 0.3s;
        }
        .eye::before {
            content: ''; position: absolute; top: 8px; left: 8px; width: 14px; height: 20px;
            background: white; border-radius: 50%; opacity: 0.9;
        }

        .nose {
            position: absolute; top: 100px; left: 50%; transform: translateX(-50%);
            width: 48px; height: 28px; background: #1b2661;
            border-radius: 50% 50% 40% 40%; z-index: 7;
            box-shadow: inset 0 2px 4px rgba(255,255,255,0.2);
        }

        .mouth {
            position: absolute; bottom: 35px; left: 50%; transform: translateX(-50%);
            width: 50px; height: 8px; border-bottom: 3px solid #1b2661;
            border-radius: 50%; z-index: 7; transition: 0.3s;
        }

        /* ЭФФЕКТЫ */
        .bad-mood .ear.left { transform: rotate(-75deg); }
        .bad-mood .ear.right { transform: rotate(75deg) scaleX(-1); }
        .sleeping .eye { height: 4px; top: 40px; }
        .sleeping .eye::before { display: none; }
        .eating .mouth { height: 20px; background: #2a1010; border-radius: 20% 20% 50% 50%; }

        /* КНОПКИ */
        .btns { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-top: 20px; }
        button {
            padding: 12px; border: none; border-radius: 15px; cursor: pointer;
            background: rgba(255, 255, 255, 0.15); color: white; font-weight: bold; 
            transition: 0.3s; border: 1px solid rgba(255,255,255,0.1);
            backdrop-filter: blur(5px);
        }
        button:hover { background: #3a56d4; transform: translateY(-3px); box-shadow: 0 5px 15px rgba(0,0,0,0.3); }
    </style>
</head>
<body>

<div id="pet-stage">
    <div class="stats-grid">
        <div class="stat-box">
            <span class="stat-label">🍖 ГОЛОД</span>
            <div class="bar-bg"><div id="h-fill" class="fill" style="background: linear-gradient(90deg, #ff5e62, #ff9966);"></div></div>
        </div>
        <div class="stat-box">
            <span class="stat-label">😊 СЧАСТЬЕ</span>
            <div class="bar-bg"><div id="ha-fill" class="fill" style="background: linear-gradient(90deg, #a8e063, #56ab2f);"></div></div>
        </div>
        <div class="stat-box">
            <span class="stat-label">⚡ ЭНЕРГИЯ</span>
            <div class="bar-bg"><div id="e-fill" class="fill" style="background: linear-gradient(90deg, #00c6ff, #0072ff);"></div></div>
        </div>
        <div class="stat-box">
            <span class="stat-label">🌈 НАСТРОЕНИЕ</span>
            <div class="bar-bg"><div id="m-fill" class="fill" style="background: linear-gradient(90deg, #f9d423, #ff4e50);"></div></div>
        </div>
    </div>

    <div id="stitch-face">
        <div class="ear left"></div>
        <div class="ear right"></div>
        <div class="head">
            <div class="eye-patch left"><div class="eye"></div></div>
            <div class="eye-patch right"><div class="eye"></div></div>
            <div class="nose"></div>
            <div id="mouth" class="mouth"></div>
        </div>
    </div>

    <h3 id="status" style="text-shadow: 0 2px 10px rgba(0,0,0,0.5);">Алоха! ✨</h3>

    <div class="btns">
        <button onclick="update('feed')">🥪 КОРМИТЬ</button>
        <button onclick="update('play')">🏄‍♂️ ИГРАТЬ</button>
        <button onclick="update('hug')">🫂 ОБНЯТЬ</button>
        <button onclick="update('sleep')">💤 СПАТЬ</button>
    </div>
</div>

<script>
    let stats = { hunger: 80, happy: 80, energy: 80, mood: 80 };
    let isSleeping = false;

    function update(action) {
        const face = document.getElementById('stitch-face');
        const msg = document.getElementById('status');

        if (isSleeping && action !== 'sleep') return;

        switch(action) {
            case 'feed':
                stats.hunger = Math.min(100, stats.hunger + 25);
                stats.mood = Math.min(100, stats.mood + 5);
                face.classList.add('eating');
                setTimeout(() => face.classList.remove('eating'), 1000);
                msg.innerText = "Стич ест сэндвич! 🥪";
                break;
            case 'play':
                if (stats.energy < 20) return msg.innerText = "Стич устал... 💤";
                stats.happy = Math.min(100, stats.happy + 20);
                stats.mood = Math.min(100, stats.mood + 15);
                stats.energy -= 25;
                msg.innerText = "Серфинг! 🏄‍♂️";
                break;
            case 'hug':
                stats.happy = Math.min(100, stats.happy + 10);
                stats.mood = Math.min(100, stats.mood + 30);
                msg.innerText = "Охана! 🫂";
                break;
            case 'sleep':
                isSleeping = !isSleeping;
                face.classList.toggle('sleeping');
                msg.innerText = isSleeping ? "Стич видит сны... 💤" : "Стич проснулся! ✨";
                break;
        }
        refreshUI();
    }

    function refreshUI() {
        document.getElementById('h-fill').style.width = stats.hunger + '%';
        document.getElementById('ha-fill').style.width = stats.happy + '%';
        document.getElementById('e-fill').style.width = stats.energy + '%';
        document.getElementById('m-fill').style.width = stats.mood + '%';

        const face = document.getElementById('stitch-face');
        if (stats.mood < 40) face.classList.add('bad-mood');
        else face.classList.remove('bad-mood');
    }

    setInterval(() => {
        if (!isSleeping) {
            stats.hunger = Math.max(0, stats.hunger - 2);
            stats.happy = Math.max(0, stats.happy - 2);
            stats.energy = Math.max(0, stats.energy - 1);
            stats.mood = Math.max(0, stats.mood - 1.5);
        } else {
            stats.energy = Math.min(100, stats.energy + 5);
            if (stats.energy >= 100) update('sleep');
        }
        refreshUI();
    }, 3000);

    refreshUI();
</script>

</body>
</html>

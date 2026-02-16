<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تحدي الذاكرة - ابدأ الآن</title>
    <style>
        :root { --primary: #8e44ad; --success: #27ae60; --danger: #e74c3c; --dark: #2c3e50; }
        body { font-family: 'Cairo', sans-serif; background: #1a1a2e; min-height: 100vh; display: flex; flex-direction: column; align-items: center; margin: 0; color: white; }
        .game-header { text-align: center; padding: 20px; background: rgba(255,255,255,0.05); width: 100%; border-bottom: 2px solid var(--primary); }
        
        /* شاشة البداية */
        #start-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; transition: 0.5s; }
        .start-btn { padding: 20px 50px; font-size: 2rem; background: var(--primary); color: white; border: none; border-radius: 50px; cursor: pointer; box-shadow: 0 0 30px var(--primary); transition: 0.3s; font-weight: bold; }
        .start-btn:hover { transform: scale(1.1); background: #9b59b6; }
        
        .levels-btn button { padding: 10px 20px; margin: 5px; border: 1px solid var(--primary); border-radius: 10px; cursor: pointer; background: transparent; color: white; transition: 0.3s; }
        button.active { background: var(--primary); }

        .grid { display: grid; gap: 15px; margin: 20px; opacity: 0; transition: opacity 0.5s; }
        .grid.visible { opacity: 1; }
        .grid.easy { grid-template-columns: repeat(4, 80px); }
        .grid.medium { grid-template-columns: repeat(4, 80px); }
        .grid.hard { grid-template-columns: repeat(6, 70px); }

        .card { width: 80px; height: 80px; position: relative; cursor: pointer; transform-style: preserve-3d; transition: transform 0.5s; }
        .grid.hard .card { width: 70px; height: 70px; }
        .card.flipped { transform: rotateY(180deg); }
        .card-front, .card-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 2rem; }
        .card-front { background: var(--dark); border: 2px solid var(--primary); }
        .card-back { background: #eee; color: #222; transform: rotateY(180deg); }
        .card.matched .card-back { background: var(--success); color: white; }

        .stats { margin-top: 15px; font-size: 1.2rem; display: flex; gap: 30px; }
        #timer.low { color: var(--danger); font-weight: bold; }
    </style>
</head>
<body>

    <!-- شاشة البداية -->
    <div id="start-overlay">
        <h1 style="margin-bottom: 30px;">جاهز للتحدي؟ 🧠</h1>
        <button class="start-btn" onclick="initGame()">إبدأ اللعب (START)</button>
        <p style="margin-top: 20px; color: #aaa;">اختر المستوى قبل البدء</p>
    </div>

    <div class="game-header">
        <h1>تحدي الذاكرة الذكي</h1>
        <div class="levels-btn">
            <button onclick="changeLevel('easy')" id="btn-easy" class="active">سهل</button>
            <button onclick="changeLevel('medium')" id="btn-medium">متوسط</button>
            <button onclick="changeLevel('hard')" id="btn-hard">صعب</button>
        </div>
        <div class="stats">
            <div>الوقت: <span id="timer">00</span>ث</div>
            <div>المحاولات: <span id="moves">0</span></div>
        </div>
    </div>

    <div class="grid" id="gameGrid"></div>

    <script>
        const symbols = ['🦁', '🐯', '🐼', '🦊', '🐨', '🐸', '🤖', '👻', '🤡', '👽', '🦄', '🐲'];
        let currentLevel = 'easy';
        let cards = [], flippedCards = [], matchedCount = 0, moves = 0, canClick = false;
        let timeLeft, timerInterval;

        function changeLevel(level) {
            currentLevel = level;
            document.querySelectorAll('.levels-btn button').forEach(b => b.classList.remove('active'));
            document.getElementById(`btn-${level}`).classList.add('active');
            document.getElementById('start-overlay').style.display = 'flex'; // إظهار شاشة البداية عند تغيير المستوى
        }

        function initGame() {
            // إخفاء شاشة البداية
            document.getElementById('start-overlay').style.display = 'none';
            const grid = document.getElementById('gameGrid');
            grid.classList.add('visible');
            
            // إعدادات المستوى
            let config = { easy: {p: 4, t: 30}, medium: {p: 8, t: 60}, hard: {p: 12, t: 100} }[currentLevel];
            timeLeft = config.t;
            moves = 0; matchedCount = 0; flippedCards = [];
            document.getElementById('moves').innerText = moves;
            document.getElementById('timer').innerText = timeLeft;

            // تجهيز الكروت
            let selected = symbols.slice(0, config.p);
            cards = [...selected, ...selected].sort(() => Math.random() - 0.5);
            
            grid.className = `grid visible ${currentLevel}`;
            grid.innerHTML = '';
            
            cards.forEach(symbol => {
                const card = document.createElement('div');
                card.className = 'card flipped'; // ميزة: الكروت تبدأ مقلوبة عشان اللاعب يشوفها ثانية
                card.innerHTML = `<div class="card-front">?</div><div class="card-back">${symbol}</div>`;
                grid.appendChild(card);
                
                // بعد ثانية واحدة، الكروت تتغطى والوقت يبدأ
                setTimeout(() => {
                    card.classList.remove('flipped');
                    card.onclick = () => flipCard(card, symbol);
                    if(cards.indexOf(symbol) === 0) { // عشان نشغل التايمر مرة واحدة بس
                        canClick = true;
                    }
                }, 1500);
            });

            setTimeout(startTimer, 1500);
        }

        function startTimer() {
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft;
                if(timeLeft <= 0) {
                    clearInterval(timerInterval);
                    alert("انتهى الوقت! حاول مرة أخرى.");
                    location.reload();
                }
            }, 1000);
        }

        function flipCard(card, symbol) {
            if(!canClick || card.classList.contains('flipped') || card.classList.contains('matched')) return;
            card.classList.add('flipped');
            flippedCards.push({card, symbol});

            if(flippedCards.length === 2) {
                moves++;
                document.getElementById('moves').innerText = moves;
                checkMatch();
            }
        }

        function checkMatch() {
            canClick = false;
            const [c1, c2] = flippedCards;
            if(c1.symbol === c2.symbol) {
                c1.card.classList.add('matched');
                c2.card.classList.add('matched');
                matchedCount += 2;
                flippedCards = [];
                canClick = true;
                if(matchedCount === cards.length) {
                    clearInterval(timerInterval);
                    setTimeout(() => alert("عاش يا وحش! فزت بالتحدي!"), 300);
                }
            } else {
                setTimeout(() => {
                    c1.card.classList.remove('flipped');
                    c2.card.classList.remove('flipped');
                    flippedCards = [];
                    canClick = true;
                }, 600);
            }
        }
    </script>
</body>
</html>

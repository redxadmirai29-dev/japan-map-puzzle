<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🌸 日本地図パズル 🌸</title>
    <style>
        :root {
            --bg-color: #fff0f5;
            --primary-color: #ff1493;
            --accent-color: #ffb6c1;
            --sea-color: #e0f7fa;
        }

        body { 
            background-color: var(--bg-color); 
            text-align: center; 
            font-family: 'Helvetica Neue', Arial, sans-serif; 
            margin: 0; 
            padding: 20px; 
            color: #333;
        }

        h1 { 
            color: var(--primary-color); 
            font-size: clamp(24px, 6vw, 32px); 
            margin: 10px 0;
            text-shadow: 1px 1px 2px white;
        }

        #ui { 
            background: white; 
            padding: 15px; 
            border: 3px solid var(--accent-color); 
            border-radius: 20px; 
            display: inline-block; 
            margin-bottom: 20px; 
            box-shadow: 0 4px 10px rgba(0,0,0,0.1); 
            width: 95%; 
            max-width: 500px; 
        }

        #target-name { 
            font-size: 32px; 
            font-weight: bold; 
            color: var(--primary-color); 
            display: block; 
            min-height: 40px;
            margin: 10px 0;
        }

        #timer { font-size: 20px; color: #555; font-weight: bold; font-family: monospace; }
        
        #game-board { 
            width: 100%; 
            max-width: 800px; 
            margin: 0 auto; 
            background: var(--sea-color); 
            border-radius: 15px; 
            border: 5px solid var(--accent-color); 
            padding: 10px; 
            min-height: 500px; 
            display: flex; 
            align-items: center; 
            justify-content: center;
            overflow: hidden;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.05);
        }
        
        /* 都道府県のスタイル設定 */
        path, polygon { 
            fill: #ffffff; 
            stroke: #999; 
            stroke-width: 0.3; 
            cursor: pointer; 
            transition: fill 0.2s, stroke 0.2s; 
        }

        path:hover, polygon:hover { fill: #fff9c4; }

        .correct { 
            fill: var(--primary-color) !important; 
            stroke: white; 
            stroke-width: 0.5;
            pointer-events: none; 
        }
        
        @keyframes miss-ani { 
            0% { fill: #ff5252; } 
            100% { fill: #ffffff; } 
        }
        .miss { animation: miss-ani 0.5s; }
        
        svg { width: 100%; height: auto; max-height: 80vh; }

        /* レスポンシブ調整 */
        @media (max-width: 600px) {
            body { padding: 10px; }
            #game-board { min-height: 350px; }
        }
    </style>
</head>
<body>

    <h1>🌸 日本地図パズル 🌸</h1>

    <div id="ui">
        <div>タイム: <span id="timer">00:00</span></div>
        <div style="margin-top:10px;">
            <span>さがしてね！</span>
            <span id="target-name">読み込み中...</span>
        </div>
    </div>

    <div id="game-board">
        <p id="loading-msg">地図データを取得しています...</p>
    </div>

    <script>
        const prefNames = ["北海道", "青森県", "岩手県", "宮城県", "秋田県", "山形県", "福島県", "茨城県", "栃木県", "群馬県", "埼玉県", "千葉県", "東京都", "神奈川県", "新潟県", "富山県", "石川県", "福井県", "山梨県", "長野県", "岐阜県", "静岡県", "愛知県", "三重県", "滋賀県", "京都府", "大阪府", "兵庫県", "奈良県", "和歌山県", "鳥取県", "島根県", "岡山県", "広島県", "山口県", "徳島県", "香川県", "愛媛県", "高知県", "福岡県", "佐賀県", "長崎県", "熊本県", "大分県", "宮崎県", "鹿児島県", "沖縄県"];

        let targetIndex = 0;
        let shuffled = [];
        let startTime;
        let isClear = false;
        let timerInterval;

        const board = document.getElementById('game-board');
        const targetText = document.getElementById('target-name');
        const timerText = document.getElementById('timer');

        async function loadMap() {
            try {
                // Geoloniaのオープンデータを利用
                const response = await fetch('https://geolonia.github.io/japanese-prefectures/map-full.svg');
                if (!response.ok) throw new Error("Network Error");
                const svgText = await response.text();
                board.innerHTML = svgText;
                initGame();
            } catch (error) {
                board.innerHTML = "<p style='color:red;'>地図の読み込みに失敗しました。</p>";
                targetText.innerText = "Error";
            }
        }

        function getPrefName(el) {
            const titleEl = el.querySelector('title') || (el.parentNode && el.parentNode.querySelector('title'));
            if (!titleEl) return "";
            return titleEl.textContent.split('/')[0].replace(/[\n\r\t]/g, "").trim();
        }

        function initGame() {
            shuffled = [...prefNames].sort(() => Math.random() - 0.5);
            startTime = Date.now();
            
            board.addEventListener('click', (e) => {
                if (isClear) return;
                const el = e.target.closest('path, polygon');
                if (!el) return;

                const clickedName = getPrefName(el);
                const targetName = shuffled[targetIndex];

                if (clickedName === targetName) {
                    const allPaths = board.querySelectorAll('path, polygon');
                    allPaths.forEach(p => {
                        if (getPrefName(p) === clickedName) {
                            p.classList.add('correct');
                        }
                    });
                    targetIndex++;
                    nextQuestion();
                } else if (clickedName !== "") {
                    el.classList.add('miss');
                    setTimeout(() => el.classList.remove('miss'), 500);
                }
            });

            nextQuestion();
            timerInterval = setInterval(updateTimer, 1000);
        }

        function nextQuestion() {
            if (targetIndex < shuffled.length) {
                targetText.innerText = shuffled[targetIndex];
            } else {
                isClear = true;
                clearInterval(timerInterval);
                targetText.innerText = "🎉 全県クリア！ 🎉";
                setTimeout(() => {
                    alert("おめでとうございます！\nタイム: " + timerText.innerText);
                }, 100);
            }
        }

        function updateTimer() {
            if (isClear || !startTime) return;
            const now = Math.floor((Date.now() - startTime) / 1000);
            const m = Math.floor(now / 60).toString().padStart(2, '0');
            const s = (now % 60).toString().padStart(2, '0');
            timerText.innerText = m + ":" + s;
        }

        loadMap();
    </script>
</body>
</html>

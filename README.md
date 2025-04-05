<!DOCTYPE html>
<html>
<head>
    <title>现代版贪吃蛇</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #2c3e50;
            color: white;
            font-family: Arial;
        }
        canvas {
            border: 3px solid #ecf0f1;
            border-radius: 5px;
            margin: 20px;
        }
        .panel {
            display: flex;
            gap: 30px;
            margin: 10px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background: #27ae60;
            border: none;
            color: white;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background: #219a52;
        }
        .controls {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 5px;
            margin: 10px;
        }
        .control-btn {
            width: 50px;
            height: 50px;
            background: #3498db;
        }
        @keyframes flash {
            0% { background-color: #2c3e50; }
            50% { background-color: #ffff00; }
            100% { background-color: #2c3e50; }
        }
        .flash {
            animation: flash 0.3s;
        }
    </style>
</head>
<body>
    <h1>现代贪吃蛇</h1>
    <div class="panel">
        <div>得分: <span id="score">0</span></div>
        <div>时间: <span id="timer">00:00</span></div>
    </div>
    <canvas id="gameCanvas" width="400" height="400"></canvas>
    <button id="restartBtn">重新开始</button>
    
    <div class="controls">
        <div></div>
        <button class="control-btn" id="upBtn">↑</button>
        <div></div>
        <button class="control-btn" id="leftBtn">←</button>
        <button class="control-btn" id="downBtn">↓</button>
        <button class="control-btn" id="rightBtn">→</button>
    </div>

    <audio id="eatSound">
        <source src="data:audio/wav;base64,//uQRAAAAWMSLwUIYAAsYkXgoQwAEaYLWfkWgAI0wWs/ItAAAGDgYtAgAyN+QWaAAihwMWm4G8QQRDiMcCBcH3Cc+CDv/7xA4Tvh9Rz/y8QADBwMWgQAZG/ILNAARQ4GLTcDeIIIhxGOBAuD7hOfBB3/94gcJ3w+o5/5eIAIAAAVwWgQAVQ2ORaIQwEMAJiDg95G4nQL7mQVWI6GwRcfsZAcsKkJvxgxEjzFUgfHoSQ9Qq7KNwqHwuB13MA4a1q/DmBrHgPcmjiGoh//EwC5nGPEmS4RcfkVKOhJf+WOgoxJclFz3kgn//dBA+ya1GhurNn8zb//9NNutNuhz31f////9vt///z+IdAEAAAK4LQIAKobHItEIYCGAExBwe8jcToF9zIKrEdDYIuP2MgOWFSE34wYiR5iqQPj0JIeoVdlG4VD4XA67mAcNa1fhzA1jwHuTRxDUQ//iYBczjHiTJcIuPyKlHQkv/LHQUYkuSi57yQT//uggfZNajQ3Vmz+Zt//+mm3Wm3Q576v////+32///5/EOgAAADmmg1rAhiuhkYuECPAhrDxwHwEa8ZyaAAAAAElFTkSuQmCC" type="audio/wav">
    </audio>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const GRID_SIZE = 20;
        const GRID_COUNT = canvas.width / GRID_SIZE;

        let snake = [
            { x: 10, y: 10 }
        ];
        let food = generateFood();
        let direction = 'right';
        let nextDirection = 'right';
        let score = 0;
        let gameRunning = false;
        let startTime = 0;
        let timerInterval;

        // 控制绑定
        const controlBtns = {
            upBtn: 'up',
            downBtn: 'down',
            leftBtn: 'left',
            rightBtn: 'right'
        };

        // 初始化游戏
        function initGame() {
            snake = [{ x: 10, y: 10 }];
            direction = 'right';
            nextDirection = 'right';
            score = 0;
            food = generateFood();
            gameRunning = true;
            startTime = Date.now();
            updateTimer();
            document.getElementById('score').textContent = score;
            
            if (timerInterval) clearInterval(timerInterval);
            timerInterval = setInterval(updateTimer, 1000);
            
            gameLoop();
        }

        // 生成食物
        function generateFood() {
            while (true) {
                const newFood = {
                    x: Math.floor(Math.random() * GRID_COUNT),
                    y: Math.floor(Math.random() * GRID_COUNT)
                };
                
                if (!snake.some(segment => 
                    segment.x === newFood.x && segment.y === newFood.y)) {
                    return newFood;
                }
            }
        }

        // 游戏主循环
        function gameLoop() {
            if (!gameRunning) return;

            direction = nextDirection;
            const head = {...snake[0]};

            switch(direction) {
                case 'up': head.y--; break;
                case 'down': head.y++; break;
                case 'left': head.x--; break;
                case 'right': head.x++; break;
            }

            // 碰撞检测
            if (head.x < 0 || head.x >= GRID_COUNT || 
                head.y < 0 || head.y >= GRID_COUNT ||
                snake.some(segment => 
                    segment.x === head.x && segment.y === head.y)) {
                gameOver();
                return;
            }

            snake.unshift(head);

            // 吃到食物
            if (head.x === food.x && head.y === food.y) {
                score += 10;
                document.getElementById('score').textContent = score;
                document.body.classList.add('flash');
                document.getElementById('eatSound').play();
                setTimeout(() => 
                    document.body.classList.remove('flash'), 300);
                food = generateFood();
            } else {
                snake.pop();
            }

            // 绘制游戏
            ctx.fillStyle = '#2c3e50';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 绘制食物
            ctx.fillStyle = '#e74c3c';
            ctx.fillRect(
                food.x * GRID_SIZE + 1, 
                food.y * GRID_SIZE + 1, 
                GRID_SIZE - 2, 
                GRID_SIZE - 2
            );

            // 绘制蛇
            snake.forEach((segment, index) => {
                ctx.fillStyle = index === 0 ? '#2ecc71' : '#27ae60';
                ctx.fillRect(
                    segment.x * GRID_SIZE + 1,
                    segment.y * GRID_SIZE + 1,
                    GRID_SIZE - 2,
                    GRID_SIZE - 2
                );
            });

            setTimeout(gameLoop, 100);
        }

        // 更新计时器
        function updateTimer() {
            const elapsed = Math.floor((Date.now() - startTime) / 1000);
            const minutes = String(Math.floor(elapsed / 60)).padStart(2, '0');
            const seconds = String(elapsed % 60).padStart(2, '0');
            document.getElementById('timer').textContent = 
                `${minutes}:${seconds}`;
        }

        // 游戏结束
        function gameOver() {
            gameRunning = false;
            clearInterval(timerInterval);
            ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = 'white';
            ctx.font = '40px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('游戏结束!', canvas.width/2, canvas.height/2);
        }

        // 事件监听
        document.addEventListener('keydown', (e) => {
            const keyMap = {
                ArrowUp: 'up',
                ArrowDown: 'down',
                ArrowLeft: 'left',
                ArrowRight: 'right'
            };
            
            if (keyMap[e.key] && isValidDirection(keyMap[e.key])) {
                nextDirection = keyMap[e.key];
            }
        });

        Object.entries(controlBtns).forEach(([id, dir]) => {
            document.getElementById(id).addEventListener('click', () => {
                if (isValidDirection(dir)) {
                    nextDirection = dir;
                }
            });
        });

        document.getElementById('restartBtn').addEventListener('click', initGame);

        // 方向有效性检查
        function isValidDirection(newDir) {
            const oppositeDirs = {
                up: 'down',
                down: 'up',
                left: 'right',
                right: 'left'
            };
            return direction !== oppositeDirs[newDir];
        }

        // 开始游戏
        initGame();
    </script>
</body>
</html>

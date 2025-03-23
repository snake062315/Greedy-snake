# Greedy-snake
The snake is very hungry, control it to find food.
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>多食物循环贪吃蛇</title>
    <style>
        body {
            font-family: "Microsoft YaHei", "PingFang SC", "SimHei", sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
            margin: 0;
            touch-action: none;
            position: relative;
            height: 100vh;
        }

        canvas {
            border: 2px solid #333;
            background: #fff;
        }

        #score {
            font-size: 24px;
            margin: 10px 0;
        }

        #restartButton {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 15px 30px;
            font-size: 20px;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0,0,0,0.2);
            transition: all 0.3s;
            z-index: 2;
        }

        .blur-effect {
            filter: blur(2px);
        }

        @media (hover: none) {
            #controls {
                display: grid;
                grid-template-columns: repeat(3, 60px);
                gap: 10px;
                margin-top: 20px;
            }
            canvas {
                width: 90vw;
                height: 90vw;
                max-width: 300px;
                max-height: 300px;
            }
            button {
                width: 60px;
                height: 60px;
                border-radius: 50%;
                font-size: 24px;
                touch-action: manipulation;
            }
        }
    </style>
</head>
<body>
    <h1>多食物循环贪吃蛇</h1>
    <div id="score">当前得分: 0</div>
    <canvas id="gameCanvas" width="300" height="300"></canvas>
    <button id="restartButton">重新开始游戏</button>
    <div id="controls">
        <button onclick="changeDirection('UP')">↑</button>
        <div></div>
        <button onclick="changeDirection('DOWN')">↓</button>
        <button onclick="changeDirection('LEFT')">←</button>
        <div></div>
        <button onclick="changeDirection('RIGHT')">→</button>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const restartButton = document.getElementById('restartButton');

        // 游戏参数
        const gridSize = 15;
        const tileCount = canvas.width / gridSize;
        const MAX_FOOD = 5;
        let score = 0;
        let gameSpeed = 150;
        let gameLoop;
        let foods = [];

        // 游戏对象
        let snake = [{x: 10, y: 10}];
        let dx = 0;
        let dy = 0;

        // 初始化食物
        generateFood();

        // 触摸控制变量
        let touchStartX = 0;
        let touchStartY = 0;
        const minSwipeDistance = 30;

        // 事件监听
        document.addEventListener('keydown', handleKeyDown);
        canvas.addEventListener('touchstart', handleTouchStart);
        canvas.addEventListener('touchmove', handleTouchMove);
        canvas.addEventListener('touchend', handleTouchEnd);
        restartButton.addEventListener('click', resetGame);
        restartButton.addEventListener('touchend', (e) => {
            e.preventDefault();
            resetGame();
        });

        startGame();

        function startGame() {
            gameLoop = setInterval(drawGame, gameSpeed);
        }

        function handleKeyDown(event) {
            const direction = getKeyDirection(event.keyCode);
            if (direction) changeDirection(direction);
        }

        function handleTouchStart(event) {
            event.preventDefault();
            touchStartX = event.touches[0].clientX;
            touchStartY = event.touches[0].clientY;
        }

        function handleTouchMove(event) {
            event.preventDefault();
        }

        function handleTouchEnd(event) {
            event.preventDefault();
            const touchEndX = event.changedTouches[0].clientX;
            const touchEndY = event.changedTouches[0].clientY;
            
            const deltaX = touchEndX - touchStartX;
            const deltaY = touchEndY - touchStartY;
            
            if (Math.abs(deltaX) > Math.abs(deltaY)) {
                if (Math.abs(deltaX) < minSwipeDistance) return;
                changeDirection(deltaX > 0 ? 'RIGHT' : 'LEFT');
            } else {
                if (Math.abs(deltaY) < minSwipeDistance) return;
                changeDirection(deltaY > 0 ? 'DOWN' : 'UP');
            }
        }

        function getKeyDirection(keyCode) {
            const keyMap = {
                37: 'LEFT', 65: 'LEFT',
                38: 'UP',   87: 'UP',
                39: 'RIGHT', 68: 'RIGHT',
                40: 'DOWN', 83: 'DOWN'
            };
            return keyMap[keyCode] || '';
        }

        function changeDirection(direction) {
            const goingUp = dy === -1;
            const goingDown = dy === 1;
            const goingRight = dx === 1;
            const goingLeft = dx === -1;

            switch(direction) {
                case 'LEFT':
                    if (!goingRight) { dx = -1; dy = 0; }
                    break;
                case 'UP':
                    if (!goingDown) { dx = 0; dy = -1; }
                    break;
                case 'RIGHT':
                    if (!goingLeft) { dx = 1; dy = 0; }
                    break;
                case 'DOWN':
                    if (!goingUp) { dx = 0; dy = 1; }
                    break;
            }
        }

        function drawGame() {
            // 计算新头部位置
            let head = {
                x: snake[0].x + dx,
                y: snake[0].y + dy
            };

            // 边界循环处理
            head.x = (head.x + tileCount) % tileCount;
            head.y = (head.y + tileCount) % tileCount;

            snake.unshift(head);

            // 多食物检测
            let eatenIndex = -1;
            foods.forEach((food, index) => {
                if(head.x === food.x && head.y === food.y) {
                    score += 10;
                    scoreElement.textContent = `当前得分: ${score}`;
                    eatenIndex = index;
                }
            });

            if(eatenIndex > -1) {
                foods.splice(eatenIndex, 1);
                generateFood();
            } else {
                snake.pop();
            }

            // 绘制画布
            ctx.fillStyle = '#fff';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 绘制蛇
            snake.forEach((segment, index) => {
                ctx.fillStyle = index === 0 ? '#2e7d32' : '#4CAF50';
                ctx.fillRect(
                    segment.x * gridSize + 1,
                    segment.y * gridSize + 1,
                    gridSize - 2,
                    gridSize - 2
                );
            });

            // 绘制多个食物
            ctx.fillStyle = '#d32f2f';
            foods.forEach(food => {
                ctx.fillRect(
                    food.x * gridSize + 1,
                    food.y * gridSize + 1,
                    gridSize - 2,
                    gridSize - 2
                );
            });

            // 碰撞检测
            if (isCollision(head)) gameOver();
        }

        function generateFood() {
            while(foods.length < MAX_FOOD) {
                const newFood = createSingleFood();
                if(isValidFoodPosition(newFood)) {
                    foods.push(newFood);
                }
            }
        }

        function createSingleFood() {
            let newFood;
            let isValid = false;
            while (!isValid) {
                newFood = {
                    x: Math.floor(Math.random() * tileCount),
                    y: Math.floor(Math.random() * tileCount)
                };
                isValid = isValidFoodPosition(newFood) && 
                    !snake.some(segment => 
                        segment.x === newFood.x && 
                        segment.y === newFood.y
                    );
            }
            return newFood;
        }

        function isValidFoodPosition(food) {
            return !foods.some(f => 
                f.x === food.x && 
                f.y === food.y
            );
        }

        function isCollision(head) {
            return snake.some((segment, index) => 
                index !== 0 && segment.x === head.x && segment.y === head.y
            );
        }

        function gameOver() {
            clearInterval(gameLoop);
            canvas.classList.add('blur-effect');
            restartButton.style.display = 'block';
        }

        function resetGame() {
            snake = [{x: 10, y: 10}];
            dx = 0;
            dy = 0;
            score = 0;
            foods = [];
            generateFood();
            scoreElement.textContent = '当前得分: 0';
            canvas.classList.remove('blur-effect');
            restartButton.style.display = 'none';
            clearInterval(gameLoop);
            startGame();
        }
    </script>
</body>
</html>

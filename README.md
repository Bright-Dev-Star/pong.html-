# pong.html-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pong Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #1a1a1a;
            font-family: Arial, sans-serif;
        }

        #gameContainer {
            text-align: center;
        }

        #scoreboard {
            color: #ffffff;
            font-size: 32px;
            margin-bottom: 20px;
            font-weight: bold;
        }

        canvas {
            background: #000;
            display: block;
            border: 3px solid #ffffff;
            cursor: none;
        }

        #instructions {
            color: #888;
            font-size: 14px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="scoreboard">Player: 0 | AI: 0</div>
        <canvas id="pong" width="800" height="400"></canvas>
        <div id="instructions">Use Mouse or Arrow Keys (↑↓) to control the left paddle</div>
    </div>

    <script>
        const canvas = document.getElementById('pong');
        const context = canvas.getContext('2d');

        // Paddle configuration
        const paddleWidth = 10;
        const paddleHeight = 100;

        // Player paddle (left)
        const player = {
            x: 10,
            y: canvas.height / 2 - paddleHeight / 2,
            width: paddleWidth,
            height: paddleHeight,
            color: '#00ff00',
            score: 0,
            speed: 0
        };

        // AI paddle (right)
        const ai = {
            x: canvas.width - paddleWidth - 10,
            y: canvas.height / 2 - paddleHeight / 2,
            width: paddleWidth,
            height: paddleHeight,
            color: '#ff0000',
            score: 0,
            speed: 4.5
        };

        // Ball configuration
        const ball = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            size: 8,
            velocityX: -5,
            velocityY: 5
        };

        // Input handling
        const keys = {};

        document.addEventListener('keydown', (event) => {
            keys[event.key] = true;
        });

        document.addEventListener('keyup', (event) => {
            keys[event.key] = false;
        });

        document.addEventListener('mousemove', (event) => {
            const rect = canvas.getBoundingClientRect();
            const mouseY = event.clientY - rect.top;
            player.y = Math.max(0, Math.min(mouseY - player.height / 2, canvas.height - player.height));
        });

        // Draw functions
        function drawPaddle(paddle) {
            context.fillStyle = paddle.color;
            context.fillRect(paddle.x, paddle.y, paddle.width, paddle.height);
        }

        function drawBall() {
            context.fillStyle = '#ffffff';
            context.beginPath();
            context.arc(ball.x, ball.y, ball.size, 0, Math.PI * 2);
            context.fill();
        }

        function drawCenterLine() {
            context.strokeStyle = '#444444';
            context.setLineDash([10, 10]);
            context.beginPath();
            context.moveTo(canvas.width / 2, 0);
            context.lineTo(canvas.width / 2, canvas.height);
            context.stroke();
            context.setLineDash([]);
        }

        function draw() {
            // Clear canvas
            context.fillStyle = '#000000';
            context.fillRect(0, 0, canvas.width, canvas.height);

            // Draw center line
            drawCenterLine();

            // Draw paddles and ball
            drawPaddle(player);
            drawPaddle(ai);
            drawBall();

            // Update scoreboard
            document.getElementById('scoreboard').innerHTML = `Player: ${player.score} | AI: ${ai.score}`;
        }

        // Update game logic
        function update() {
            // Handle player input
            if (keys['ArrowUp'] || keys['w']) {
                player.y = Math.max(0, player.y - 7);
            }
            if (keys['ArrowDown'] || keys['s']) {
                player.y = Math.min(canvas.height - player.height, player.y + 7);
            }

            // Update ball position
            ball.x += ball.velocityX;
            ball.y += ball.velocityY;

            // Ball collision with top and bottom walls
            if (ball.y - ball.size < 0 || ball.y + ball.size > canvas.height) {
                ball.velocityY = -ball.velocityY;
                ball.y = Math.max(ball.size, Math.min(canvas.height - ball.size, ball.y));
            }

            // Ball collision with player paddle
            if (
                ball.x - ball.size < player.x + player.width &&
                ball.y > player.y &&
                ball.y < player.y + player.height
            ) {
                ball.velocityX = -ball.velocityX;
                ball.x = player.x + player.width + ball.size;

                // Add spin based on where ball hits paddle
                let deltaY = ball.y - (player.y + player.height / 2);
                ball.velocityY = deltaY * 0.1;
            }

            // Ball collision with AI paddle
            if (
                ball.x + ball.size > ai.x &&
                ball.y > ai.y &&
                ball.y < ai.y + ai.height
            ) {
                ball.velocityX = -ball.velocityX;
                ball.x = ai.x - ball.size;

                // Add spin based on where ball hits paddle
                let deltaY = ball.y - (ai.y + ai.height / 2);
                ball.velocityY = deltaY * 0.1;
            }

            // Ball out of bounds - scoring
            if (ball.x - ball.size < 0) {
                ai.score++;
                resetBall();
            } else if (ball.x + ball.size > canvas.width) {
                player.score++;
                resetBall();
            }

            // AI paddle logic
            let aiCenter = ai.y + ai.height / 2;
            if (aiCenter < ball.y - 35) {
                ai.y = Math.min(canvas.height - ai.height, ai.y + ai.speed);
            } else if (aiCenter > ball.y + 35) {
                ai.y = Math.max(0, ai.y - ai.speed);
            }
        }

        function resetBall() {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.velocityX = (Math.random() > 0.5 ? 1 : -1) * 5;
            ball.velocityY = (Math.random() - 0.5) * 8;
        }

        // Game loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Start the game
        gameLoop();
    </script>
</body>
</html>

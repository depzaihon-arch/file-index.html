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
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            font-family: 'Arial', sans-serif;
            color: white;
        }

        .container {
            text-align: center;
        }

        h1 {
            margin-bottom: 20px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        canvas {
            display: block;
            border: 3px solid white;
            background-color: #000;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
            margin-bottom: 20px;
        }

        .instructions {
            background: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 8px;
            max-width: 600px;
            font-size: 14px;
            line-height: 1.6;
        }

        .instructions h2 {
            margin-bottom: 10px;
            font-size: 1.2em;
        }

        .controls {
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
            flex-wrap: wrap;
        }

        .control-item {
            margin: 5px 10px;
        }

        .control-item strong {
            color: #ffeb3b;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎮 PONG GAME 🎮</h1>
        <canvas id="pongCanvas" width="800" height="400"></canvas>
        <div class="instructions">
            <h2>How to Play:</h2>
            <div class="controls">
                <div class="control-item">
                    <strong>Mouse Movement:</strong> Move your paddle vertically
                </div>
                <div class="control-item">
                    <strong>↑ ↓ Arrow Keys:</strong> Alternative paddle control
                </div>
            </div>
            <p style="margin-top: 10px;">First player to score wins! The AI opponent will challenge you.</p>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('pongCanvas');
        const ctx = canvas.getContext('2d');

        // Game constants
        const GAME_WIDTH = canvas.width;
        const GAME_HEIGHT = canvas.height;
        const PADDLE_HEIGHT = 80;
        const PADDLE_WIDTH = 10;
        const BALL_RADIUS = 8;
        const PADDLE_SPEED = 5;
        const AI_SPEED = 3.5;
        const INITIAL_BALL_SPEED = 3;

        // Ball object
        const ball = {
            x: GAME_WIDTH / 2,
            y: GAME_HEIGHT / 2,
            dx: INITIAL_BALL_SPEED,
            dy: INITIAL_BALL_SPEED,
            radius: BALL_RADIUS
        };

        // Player paddle (left)
        const player = {
            x: 10,
            y: (GAME_HEIGHT - PADDLE_HEIGHT) / 2,
            width: PADDLE_WIDTH,
            height: PADDLE_HEIGHT,
            dy: 0,
            keys: {
                up: false,
                down: false
            }
        };

        // AI paddle (right)
        const ai = {
            x: GAME_WIDTH - PADDLE_WIDTH - 10,
            y: (GAME_HEIGHT - PADDLE_HEIGHT) / 2,
            width: PADDLE_WIDTH,
            height: PADDLE_HEIGHT
        };

        // Score
        let playerScore = 0;
        let aiScore = 0;

        // Keyboard controls
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowUp') {
                player.keys.up = true;
            }
            if (e.key === 'ArrowDown') {
                player.keys.down = true;
            }
        });

        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowUp') {
                player.keys.up = false;
            }
            if (e.key === 'ArrowDown') {
                player.keys.down = false;
            }
        });

        // Mouse controls
        document.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            const mouseY = e.clientY - rect.top;
            
            // Smooth mouse following
            if (mouseY > 0 && mouseY < GAME_HEIGHT) {
                player.y = mouseY - PADDLE_HEIGHT / 2;
            }
        });

        // Keep paddle in bounds
        function constrainPaddle(paddle) {
            if (paddle.y < 0) {
                paddle.y = 0;
            }
            if (paddle.y + paddle.height > GAME_HEIGHT) {
                paddle.y = GAME_HEIGHT - paddle.height;
            }
        }

        // Update player paddle position
        function updatePlayer() {
            if (player.keys.up) {
                player.y -= PADDLE_SPEED;
            }
            if (player.keys.down) {
                player.y += PADDLE_SPEED;
            }
            constrainPaddle(player);
        }

        // AI opponent logic
        function updateAI() {
            const aiCenter = ai.y + ai.height / 2;
            const ballCenter = ball.y;
            const difference = ballCenter - aiCenter;

            // Add some difficulty variation - AI isn't perfect
            const aiAccuracy = 0.85;
            
            if (difference < -35) {
                ai.y -= AI_SPEED;
            } else if (difference > 35) {
                ai.y += AI_SPEED;
            } else if (Math.random() < aiAccuracy) {
                if (difference < 0) {
                    ai.y -= AI_SPEED * 0.7;
                } else {
                    ai.y += AI_SPEED * 0.7;
                }
            }
            
            constrainPaddle(ai);
        }

        // Collision detection
        function checkPaddleCollision(paddle) {
            if (
                ball.x - ball.radius < paddle.x + paddle.width &&
                ball.x + ball.radius > paddle.x &&
                ball.y - ball.radius < paddle.y + paddle.height &&
                ball.y + ball.radius > paddle.y
            ) {
                // Calculate collision response
                const paddleCenter = paddle.y + paddle.height / 2;
                const ballCenter = ball.y;
                const distance = ballCenter - paddleCenter;
                
                // Bounce angle based on where ball hits paddle
                ball.dy = distance * 0.08;
                
                // Ensure ball moves away from paddle
                if (ball.x < GAME_WIDTH / 2) {
                    ball.dx = Math.abs(ball.dx);
                } else {
                    ball.dx = -Math.abs(ball.dx);
                }

                // Increase ball speed slightly with each paddle hit
                ball.dx *= 1.02;
                
                return true;
            }
            return false;
        }

        // Update ball position
        function updateBall() {
            ball.x += ball.dx;
            ball.y += ball.dy;

            // Wall collision (top and bottom)
            if (ball.y - ball.radius < 0) {
                ball.y = ball.radius;
                ball.dy = -ball.dy;
            }
            if (ball.y + ball.radius > GAME_HEIGHT) {
                ball.y = GAME_HEIGHT - ball.radius;
                ball.dy = -ball.dy;
            }

            // Paddle collision
            checkPaddleCollision(player);
            checkPaddleCollision(ai);

            // Score points
            if (ball.x - ball.radius < 0) {
                aiScore++;
                resetBall();
            }
            if (ball.x + ball.radius > GAME_WIDTH) {
                playerScore++;
                resetBall();
            }
        }

        // Reset ball to center
        function resetBall() {
            ball.x = GAME_WIDTH / 2;
            ball.y = GAME_HEIGHT / 2;
            const angle = (Math.random() - 0.5) * Math.PI / 4;
            ball.dx = INITIAL_BALL_SPEED * Math.cos(angle) * (Math.random() > 0.5 ? 1 : -1);
            ball.dy = INITIAL_BALL_SPEED * Math.sin(angle);
        }

        // Drawing functions
        function drawRectangle(x, y, width, height, color = 'white') {
            ctx.fillStyle = color;
            ctx.fillRect(x, y, width, height);
        }

        function drawCircle(x, y, radius, color = 'white') {
            ctx.fillStyle = color;
            ctx.beginPath();
            ctx.arc(x, y, radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.closePath();
        }

        function drawText(text, x, y, size = '20px', color = 'white') {
            ctx.fillStyle = color;
            ctx.font = `${size} Arial`;
            ctx.textAlign = 'center';
            ctx.fillText(text, x, y);
        }

        function drawCenterLine() {
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.2)';
            ctx.setLineDash([10, 10]);
            ctx.beginPath();
            ctx.moveTo(GAME_WIDTH / 2, 0);
            ctx.lineTo(GAME_WIDTH / 2, GAME_HEIGHT);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        function draw() {
            // Clear canvas
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

            // Draw center line
            drawCenterLine();

            // Draw paddles
            drawRectangle(player.x, player.y, player.width, player.height, '#00ff00');
            drawRectangle(ai.x, ai.y, ai.width, ai.height, '#ff0000');

            // Draw ball
            drawCircle(ball.x, ball.y, ball.radius, '#ffeb3b');

            // Draw score
            drawText(`${playerScore}`, GAME_WIDTH / 4, 40, '32px', '#00ff00');
            drawText(`${aiScore}`, (GAME_WIDTH * 3) / 4, 40, '32px', '#ff0000');

            // Update game state
            updatePlayer();
            updateAI();
            updateBall();

            // Continue animation
            requestAnimationFrame(draw);
        }

        // Start the game
        draw();
    </script>
</body>
</html>

<get the most point>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Space Defender</title>
    <style>
        :root {
            --neon-blue: #00f3ff;
            --neon-pink: #ff00ff;
            --neon-green: #00ff41;
            --bg-color: #050510;
            --ui-bg: rgba(20, 20, 40, 0.9);
            --font-main: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        * {
            box-sizing: border-box;
            touch-action: none; /* Prevents zooming/scrolling on mobile while playing */
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: #fff;
            font-family: var(--font-main);
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        /* --- UI Overlay --- */
        #game-ui {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none; /* Let clicks pass through to canvas when playing */
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            z-index: 10;
        }

        .hud-top {
            padding: 20px;
            display: flex;
            justify-content: space-between;
            font-size: 1.2rem;
            text-shadow: 0 0 10px var(--neon-blue);
            font-weight: bold;
        }

        /* --- Screens (Start / Game Over) --- */
        .screen {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: var(--ui-bg);
            border: 2px solid var(--neon-blue);
            box-shadow: 0 0 20px var(--neon-blue);
            padding: 40px;
            text-align: center;
            border-radius: 10px;
            pointer-events: auto;
            display: flex;
            flex-direction: column;
            gap: 20px;
            min-width: 300px;
            backdrop-filter: blur(5px);
            transition: opacity 0.3s ease;
        }

        .hidden {
            opacity: 0;
            pointer-events: none;
            display: none !important;
        }

        h1 {
            margin: 0;
            font-size: 2.5rem;
            color: var(--neon-blue);
            text-transform: uppercase;
            letter-spacing: 2px;
            text-shadow: 0 0 15px var(--neon-blue);
        }

        p {
            color: #ccc;
            line-height: 1.5;
        }

        .btn {
            background: transparent;
            color: var(--neon-green);
            border: 2px solid var(--neon-green);
            padding: 12px 24px;
            font-size: 1.2rem;
            cursor: pointer;
            text-transform: uppercase;
            font-weight: bold;
            transition: all 0.2s ease;
            box-shadow: 0 0 10px var(--neon-green);
            border-radius: 4px;
        }

        .btn:hover {
            background: var(--neon-green);
            color: #000;
            box-shadow: 0 0 25px var(--neon-green);
        }

        /* --- Canvas --- */
        canvas {
            background: radial-gradient(circle at center, #1a1a2e 0%, #000 100%);
            box-shadow: 0 0 30px rgba(0, 243, 255, 0.2);
            max-width: 100%;
            max-height: 100%;
            cursor: crosshair;
        }

        /* --- Mobile Controls --- */
        #mobile-controls {
            display: none; /* Hidden on desktop by default, shown via JS or media query if needed */
            width: 100%;
            padding: 20px;
            padding-bottom: 40px;
            justify-content: space-between;
            pointer-events: auto;
        }

        .control-btn {
            width: 70px;
            height: 70px;
            border-radius: 50%;
            border: 2px solid rgba(255, 255, 255, 0.3);
            background: rgba(255, 255, 255, 0.1);
            color: white;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            user-select: none;
        }
        
        .control-btn:active {
            background: rgba(0, 243, 255, 0.3);
            border-color: var(--neon-blue);
        }

        .d-pad {
            display: flex;
            gap: 15px;
        }

        /* Show mobile controls on small screens */
        @media (max-width: 768px) {
            #mobile-controls {
                display: flex;
            }
            h1 { font-size: 1.8rem; }
            .screen { padding: 20px; width: 90%; }
        }
    </style>
</head>
<body>

    <!-- UI Layer -->
    <div id="game-ui">
        <div class="hud-top">
            <div id="score-display">SCORE: 0</div>
            <div id="lives-display">LIVES: 3</div>
        </div>

        <!-- Start Screen -->
        <div id="start-screen" class="screen">
            <h1>Neon Defender</h1>
            <p>Defend the grid from incoming enemies.</p>
            <p style="font-size: 0.9rem; color: #888;">
                Desktop: Arrow Keys to move, Space to shoot.<br>
                Mobile: Use on-screen buttons.
            </p>
            <button class="btn" id="start-btn">Start Mission</button>
        </div>

        <!-- Game Over Screen -->
        <div id="game-over-screen" class="screen hidden">
            <h1 style="color: var(--neon-pink); text-shadow: 0 0 15px var(--neon-pink);">Game Over</h1>
            <p>Final Score: <span id="final-score" style="color: white; font-weight: bold;">0</span></p>
            <button class="btn" id="restart-btn">Try Again</button>
        </div>

        <!-- Mobile Controls -->
        <div id="mobile-controls">
            <div class="d-pad">
                <div class="control-btn" id="btn-left">←</div>
                <div class="control-btn" id="btn-right">→</div>
            </div>
            <div class="control-btn" id="btn-fire" style="border-color: var(--neon-pink);">🔥</div>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        /** 
         * NEON SPACE DEFENDER
         * A lightweight HTML5 Canvas Shooter
         */

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // UI Elements
        const startScreen = document.getElementById('start-screen');
        const gameOverScreen = document.getElementById('game-over-screen');
        const scoreEl = document.getElementById('score-display');
        const livesEl = document.getElementById('lives-display');
        const finalScoreEl = document.getElementById('final-score');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');

        // Mobile Controls
        const btnLeft = document.getElementById('btn-left');
        const btnRight = document.getElementById('btn-right');
        const btnFire = document.getElementById('btn-fire');

        // Game State
        let gameRunning = false;
        let score = 0;
        let lives = 3;
        let frames = 0;
        let difficultyMultiplier = 1;

        // Input State
        const keys = {
            ArrowLeft: false,
            ArrowRight: false,
            Space: false
        };

        // Resize Handling
        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // --- Game Entities ---

        // 1. Player Ship
        const player = {
            x: canvas.width / 2,
            y: canvas.height - 80,
            width: 40,
            height: 40,
            speed: 7,
            color: '#00f3ff',
            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                
                // Ship Body (Triangle)
                ctx.beginPath();
                ctx.moveTo(0, -this.height / 2);
                ctx.lineTo(this.width / 2, this.height / 2);
                ctx.lineTo(0, this.height / 2 - 10); // Indent at bottom
                ctx.lineTo(-this.width / 2, this.height / 2);
                ctx.closePath();
                
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 20;
                ctx.shadowColor = this.color;
                ctx.fill();

                // Engine Flame (Flicker effect)
                if (gameRunning) {
                    ctx.beginPath();
                    ctx.moveTo(-5, this.height / 2 - 5);
                    ctx.lineTo(5, this.height / 2 - 5);
                    ctx.lineTo(0, this.height / 2 + 10 + Math.random() * 10);
                    ctx.fillStyle = '#ff00ff';
                    ctx.shadowColor = '#ff00ff';
                    ctx.fill();
                }

                ctx.restore();
            },
            update() {
                // Movement
                if (keys.ArrowLeft && this.x > this.width) {
                    this.x -= this.speed;
                }
                if (keys.ArrowRight && this.x < canvas.width - this.width) {
                    this.x += this.speed;
                }
            }
        };

        // 2. Bullets
        let bullets = [];
        class Bullet {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = 4;
                this.speed = 10;
                this.color = '#fff';
                this.markedForDeletion = false;
            }
            update() {
                this.y -= this.speed;
                if (this.y < 0) this.markedForDeletion = true;
            }
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fill();
                ctx.closePath();
            }
        }

        // 3. Enemies
        let enemies = [];
        class Enemy {
            constructor() {
                this.radius = 15 + Math.random() * 15; // Variable size
                this.x = Math.random() * (canvas.width - this.radius * 2) + this.radius;
                this.y = -this.radius;
                this.speed = (2 + Math.random() * 2) * difficultyMultiplier;
                this.color = '#ff004c';
                this.markedForDeletion = false;
                // Slight wobble movement
                this.angle = 0;
                this.angleSpeed = Math.random() * 0.1 - 0.05;
            }
            update() {
                this.y += this.speed;
                this.x += Math.sin(this.angle) * 2;
                this.angle += this.angleSpeed;

                if (this.y > canvas.height + this.radius) {
                    this.markedForDeletion = true;
                    // Penalty for missing an enemy? Let's say no penalty for now, just lost opportunity.
                }
            }
            draw() {
                ctx.beginPath();
                // Draw a Hexagon-ish shape or jagged circle
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.fill();
                
                // Inner core
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius * 0.5, 0, Math.PI * 2);
                ctx.fillStyle = '#fff';
                ctx.fill();
            }
        }

        // 4. Particles (Explosions)
        let particles = [];
        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.size = Math.random() * 3 + 1;
                this.speedX = Math.random() * 6 - 3;
                this.speedY = Math.random() * 6 - 3;
                this.color = color;
                this.life = 1.0; // Opacity
                this.decay = 0.03;
            }
            update() {
                this.x += this.speedX;
                this.y += this.speedY;
                this.life -= this.decay;
            }
            draw() {
                ctx.save();
                ctx.globalAlpha = this.life;
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }
        }

        // 5. Stars (Background)
        let stars = [];
        function initStars() {
            stars = [];
            for(let i=0; i<100; i++) {
                stars.push({
                    x: Math.random() * canvas.width,
                    y: Math.random() * canvas.height,
                    size: Math.random() * 2,
                    speed: Math.random() * 0.5 + 0.1
                });
            }
        }
        function drawBackground() {
            ctx.fillStyle = '#050510';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = '#ffffff';
            stars.forEach(star => {
                ctx.globalAlpha = Math.random() * 0.5 + 0.3;
                ctx.fillRect(star.x, star.y, star.size, star.size);
                star.y += star.speed * (gameRunning ? 5 : 1); // Move faster when playing
                if(star.y > canvas.height) {
                    star.y = 0;
                    star.x = Math.random() * canvas.width;
                }
            });
            ctx.globalAlpha = 1.0;
        }

        // --- Core Functions ---

        function spawnEnemy() {
            // Spawn rate increases slightly as score goes up
            let spawnRate = 60; // frames
            if (score > 500) spawnRate = 50;
            if (score > 1000) spawnRate = 40;
            if (score > 2000) spawnRate = 30;

            if (frames % spawnRate === 0) {
                enemies.push(new Enemy());
            }
        }

        function createExplosion(x, y, color) {
            for (let i = 0; i < 15; i++) {
                particles.push(new Particle(x, y, color));
            }
        }

        function checkCollisions() {
            // Bullet vs Enemy
            bullets.forEach(bullet => {
                enemies.forEach(enemy => {
                    const dist = Math.hypot(bullet.x - enemy.x, bullet.y - enemy.y);
                    
                    if (dist - enemy.radius - bullet.radius < 1) {
                        // Hit!
                        createExplosion(enemy.x, enemy.y, enemy.color);
                        enemy.markedForDeletion = true;
                        bullet.markedForDeletion = true;
                        score += 100;
                        scoreEl.innerText = `SCORE: ${score}`;
                        
                        // Increase difficulty slightly
                        if(score % 1000 === 0) difficultyMultiplier += 0.2;
                    }
                });
            });

            // Enemy vs Player
            enemies.forEach(enemy => {
                // Simple box/circle collision approximation
                const dist = Math.hypot(player.x - enemy.x, player.y - enemy.y);
                if (dist - enemy.radius - player.width/2 < 0) {
                    enemy.markedForDeletion = true;
                    createExplosion(player.x, player.y, '#ff0000');
                    lives--;
                    livesEl.innerText = `LIVES: ${lives}`;
                    
                    // Screen shake effect
                    canvas.style.transform = `translate(${Math.random()*10-5}px, ${Math.random()*10-5}px)`;
                    setTimeout(() => canvas.style.transform = 'none', 100);

                    if (lives <= 0) {
                        endGame();
                    }
                }
            });
        }

        function gameLoop() {
            if (!gameRunning) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawBackground();

            // Player
            player.update();
            player.draw();

            // Bullets
            bullets.forEach((b, index) => {
                b.update();
                b.draw();
                if (b.markedForDeletion) bullets.splice(index, 1);
            });

            // Enemies
            spawnEnemy();
            enemies.forEach((e, index) => {
                e.update();
                e.draw();
                if (e.markedForDeletion) enemies.splice(index, 1);
            });

            // Particles
            particles.forEach((p, index) => {
                p.update();
                p.draw();
                if (p.life <= 0) particles.splice(index, 1);
            });

            checkCollisions();

            frames++;
            requestAnimationFrame(gameLoop);
        }

        function initGame() {
            // Reset variables
            score = 0;
            lives = 3;
            difficultyMultiplier = 1;
            frames = 0;
            bullets = [];
            enemies = [];
            particles = [];
            
            // Update UI
            scoreEl.innerText = "SCORE: 0";
            livesEl.innerText = "LIVES: 3";
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            
            // Reset Player Position
            player.x = canvas.width / 2;
            player.y = canvas.height - 80;

            gameRunning = true;
            initStars(); // Reset stars
            gameLoop();
        }

        function endGame() {
            gameRunning = false;
            finalScoreEl.innerText = score;
            gameOverScreen.classList.remove('hidden');
        }

        // --- Input Listeners ---

        // Keyboard
        window.addEventListener('keydown', e => {
            if (e.code === 'ArrowLeft') keys.ArrowLeft = true;
            if (e.code === 'ArrowRight') keys.ArrowRight = true;
            if (e.code === 'Space') {
                if (!keys.Space && gameRunning) {
                    bullets.push(new Bullet(player.x, player.y - player.height/2));
                }
                keys.Space = true;
            }
        });

        window.addEventListener('keyup', e => {
            if (e.code === 'ArrowLeft') keys.ArrowLeft = false;
            if (e.code === 'ArrowRight') keys.ArrowRight = false;
            if (e.code === 'Space') keys.Space = false;
        });

        // Mobile Touch (Simulated Key Presses)
        const addTouch = (elem, code) => {
            elem.addEventListener('touchstart', (e) => {
                e.preventDefault();
                if (code === 'Space') {
                    if(gameRunning) bullets.push(new Bullet(player.x, player.y - player.height/2));
                } else {
                    keys[code] = true;
                }
                elem.style.background = 'rgba(0, 243, 255, 0.3)';
            });
            elem.addEventListener('touchend', (e) => {
                e.preventDefault();
                if (code !== 'Space') keys[code] = false;
                elem.style.background = 'rgba(255, 255, 255, 0.1)';
            });
        };

        addTouch(btnLeft, 'ArrowLeft');
        addTouch(btnRight, 'ArrowRight');
        addTouch(btnFire, 'Space');

        // UI Buttons
        startBtn.addEventListener('click', initGame);
        restartBtn.addEventListener('click', initGame);

        // Initial Draw (Menu Background)
        initStars();
        function attractLoop() {
            if(!gameRunning) {
                drawBackground();
                requestAnimationFrame(attractLoop);
            }
        }
        attractLoop();

    </script>
</body>
</html>

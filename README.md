<!DOCTYPE html>
<html lang="pt">
<head>
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

<link rel="apple-touch-icon" href="https://via.placeholder.com/192">

    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Gravity Flip Arcade</title>
    <style>
        body { margin: 0; overflow: hidden; background: #0a0a0a; font-family: 'Courier New', Courier, monospace; }
        canvas { display: block; }
        #ui { 
            position: absolute; top: 20px; left: 20px; color: #00ffcc; 
            font-size: 28px; font-weight: bold; text-shadow: 0 0 10px #00ffcc;
            pointer-events: none; 
        }
    </style>
</head>
<body>

    <div id="ui">SCORE: 000</div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const ui = document.getElementById('ui');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // --- Configurações de Jogo ---
        let gameActive = true;
        let score = 0;
        let gameSpeed = 4; // Velocidade inicial lenta
        const maxSpeed = 15; // Limite de velocidade
        const acceleration = 0.001; // Aceleração constante por frame

        // --- Jogador com Física Melhorada ---
        let player = {
            x: canvas.width * 0.15,
            y: canvas.height / 2,
            size: 25,
            targetGravity: 0.8, // Para onde a gravidade quer ir
            currentGravity: 0.8,
            velocity: 0,
            friction: 0.98,
            trail: [] // Para o efeito visual de rasto
        };

        function flipGravity() {
            if (!gameActive) return;
            player.targetGravity *= -1;
        }

        // Controlos
        window.addEventListener('mousedown', flipGravity);
        window.addEventListener('touchstart', (e) => { e.preventDefault(); flipGravity(); });

        // --- Obstáculos Estilizados ---
        let obstacles = [];
        function spawnObstacle() {
            if (!gameActive) return;
            const margin = 50;
            let height = Math.random() * (canvas.height * 0.4) + 40;
            let isTop = Math.random() > 0.5;
            
            obstacles.push({
                x: canvas.width + 50,
                y: isTop ? 0 : canvas.height - height,
                w: 30,
                h: height,
                hue: Math.random() * 360 // Cores variadas para os obstáculos
            });
            
            // O tempo entre obstáculos diminui conforme a velocidade aumenta
            let nextSpawn = Math.max(500, 1800 - (gameSpeed * 100));
            setTimeout(spawnObstacle, nextSpawn);
        }

        function update() {
            if (!gameActive) return;

            // 1. Aceleração do Jogo (Fica mais rápido com o tempo)
            if (gameSpeed < maxSpeed) gameSpeed += acceleration;

            // 2. Física Suave (Suaviza a mudança de gravidade)
            // A gravidade atual "persegue" a gravidade alvo
            player.currentGravity += (player.targetGravity - player.currentGravity) * 0.1;
            player.velocity += player.currentGravity;
            player.y += player.velocity;

            // 3. Efeito de Rasto (Trail)
            player.trail.push({x: player.x, y: player.y});
            if (player.trail.length > 10) player.trail.shift();

            // 4. Limites de Ecrã
            if (player.y < 0) { player.y = 0; player.velocity = 0; }
            if (player.y + player.size > canvas.height) { 
                player.y = canvas.height - player.size; 
                player.velocity = 0; 
            }

            // 5. Obstáculos e Colisões
            obstacles.forEach((obs, index) => {
                obs.x -= gameSpeed;

                // Colisão AABB simples
                if (player.x < obs.x + obs.w &&
                    player.x + player.size > obs.x &&
                    player.y < obs.y + obs.h &&
                    player.y + player.size > obs.y) {
                    gameOver();
                }

                if (obs.x + obs.w < 0) {
                    obstacles.splice(index, 1);
                    score++;
                    ui.innerText = "SCORE: " + score.toString().padStart(3, '0');
                }
            });

            draw();
            requestAnimationFrame(update);
        }

        function draw() {
            // Fundo com rasto de movimento
            ctx.fillStyle = 'rgba(10, 10, 10, 0.3)'; 
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Desenhar Rasto do Jogador
            player.trail.forEach((t, i) => {
                ctx.fillStyle = `rgba(0, 255, 204, ${i / 10})`;
                ctx.fillRect(t.x, t.y, player.size, player.size);
            });

            // Desenhar Jogador (Brilho Neon)
            ctx.shadowBlur = 15;
            ctx.shadowColor = '#00ffcc';
            ctx.fillStyle = '#00ffcc';
            ctx.fillRect(player.x, player.y, player.size, player.size);
            ctx.shadowBlur = 0; // Reset para os outros elementos

            // Desenhar Obstáculos com Gradiente
            obstacles.forEach(obs => {
                let grad = ctx.createLinearGradient(obs.x, obs.y, obs.x + obs.w, obs.y + obs.h);
                grad.addColorStop(0, `hsl(${obs.hue}, 100%, 50%)`);
                grad.addColorStop(1, '#000');
                ctx.fillStyle = grad;
                ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
            });
        }

        function gameOver() {
            gameActive = false;
            ctx.fillStyle = "rgba(0,0,0,0.7)";
            ctx.fillRect(0,0,canvas.width, canvas.height);
            ctx.fillStyle = "white";
            ctx.textAlign = "center";
            ctx.font = "40px Arial";
            ctx.fillText("GAME OVER", canvas.width/2, canvas.height/2);
            ctx.font = "20px Arial";
            ctx.fillText("Toca para reiniciar", canvas.width/2, canvas.height/2 + 50);
            
            window.onclick = () => location.reload();
        }

        spawnObstacle();
        update();
    </script>
</body>
</html>


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flying Bird Game</title>
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { text-align: center; font-family: 'Playfair Display', serif; overflow: hidden; }
        canvas { display: block; margin: auto; background: url('https://i.postimg.cc/jjnD5Jcz/images-1.jpg') no-repeat center/cover; }
        
        #start, #restart, #finalScore {
            position: absolute; left: 50%; transform: translateX(-50%);
            font-family: 'Playfair Display', serif; padding: 16px 32px;
            font-size: 28px; border-radius: 10px; border: 3px solid #B8860B;
            cursor: pointer; font-weight: bold;
        }

        #start {
            top: 45%; background: linear-gradient(45deg, #FFD700, #DAA520);
            color: white; box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3); 
        }

        #finalScore {
            display: none; 
            top: 35%;
            font-size: 32px;
            background: rgba(0, 0, 0, 0.8);
            color: gold;
            padding: 16px 32px;
            border-radius: 12px;
            letter-spacing: 2px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.5);
        }

        #restart {
            display: none; 
            background: linear-gradient(45deg, #DAA520, #FFD700);
            color: white; 
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
            top: 50%;  /* Changed to position in center vertically */
            transform: translateY(-50%); /* Center vertically */
            position: absolute;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <button id="start" onclick="startGame()">Start Game</button>
    <div id="finalScore"></div>
    <button id="restart" onclick="restartGame()">Restart</button>

    <audio id="restartSound" src="https://www.soundjay.com/button/beep-07.wav"></audio> <!-- Add pop sound effect here -->

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        canvas.width = window.innerWidth;  /* Set canvas width to window's width */
        canvas.height = window.innerHeight;  /* Set canvas height to window's height */

        // ✅ **Image Load**
        const birdImg = new Image();
        birdImg.src = "https://i.postimg.cc/bJN9Z3Vp/Bird.png";

        const pipeImg = new Image();
        pipeImg.src = "https://i.postimg.cc/pdBBVgZs/Pipe.png";

        let bird, pipes, score, gameRunning, gravity;

        function initializeGame() {
            bird = { x: 120, y: canvas.height / 2, width: 50, height: 50, velocity: 0, lift: -10 };  /* Adjusted jump force */
            pipes = [];
            score = 0;
            gameRunning = false;
            gravity = 0.4;
        }

        function drawBird() {
            ctx.drawImage(birdImg, bird.x, bird.y, bird.width, bird.height);
        }

        function drawPipes() {
            for (let i = pipes.length - 1; i >= 0; i--) {
                let { x: pipeX, y: pipeY, scored } = pipes[i];
                let gap = 200, pipeWidth = 100, pipeHeight = 500;  /* Changed pipe height from 600 to 500 */

                ctx.drawImage(pipeImg, pipeX, pipeY, pipeWidth, pipeHeight);
                ctx.drawImage(pipeImg, pipeX, pipeY + pipeHeight + gap, pipeWidth, pipeHeight);

                pipes[i].x -= 4.5;

                if (pipeX + pipeWidth < 0) pipes.splice(i, 1);

                if (!scored && bird.x > pipeX + pipeWidth) {
                    score++;
                    pipes[i].scored = true;
                }

                if (
                    (bird.x + bird.width > pipeX && bird.x < pipeX + pipeWidth &&
                        (bird.y < pipeY + pipeHeight || bird.y + bird.height > pipeY + pipeHeight + gap)) ||
                    bird.y + bird.height >= canvas.height
                ) {
                    gameOver();
                }
            }
        }

        function drawScore() {
            ctx.fillStyle = "#FFF";
            ctx.strokeStyle = "#000";
            ctx.lineWidth = 4;
            ctx.font = "32px 'Playfair Display', serif";  /* Adjusted score font size */
            ctx.strokeText("Score: " + score, 20, 40);
            ctx.fillText("Score: " + score, 20, 40);
        }

        function spawnPipes() {
            if (gameRunning && (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 300)) {
                let pipeY = Math.random() * (-300) - 100;
                pipes.push({ x: canvas.width, y: pipeY, scored: false });
            }
            setTimeout(spawnPipes, 1500);  /* Increased pipe spawn interval */
        }

        function gameLoop() {
            if (!gameRunning) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawBird();
            drawPipes();
            drawScore();
            bird.velocity += gravity;
            bird.y += bird.velocity;

            // Smooth the bird's fall
            if (bird.y > canvas.height - bird.height) bird.y = canvas.height - bird.height;  // Prevent going off-screen

            requestAnimationFrame(gameLoop);
        }

        function startGame() {
            document.getElementById("start").style.display = "none";
            document.getElementById("restart").style.display = "none";
            document.getElementById("finalScore").style.display = "none";
            initializeGame();
            gameRunning = true;
            spawnPipes();
            gameLoop();
        }

        function gameOver() {
            gameRunning = false;

            let finalScoreDiv = document.getElementById("finalScore");
            finalScoreDiv.innerText = `Game Over\nFinal Score: ${score}`;
            finalScoreDiv.style.display = "block";

            let restartButton = document.getElementById("restart");
            restartButton.style.display = "block";  
        }

        function restartGame() {
            // Play restart sound on click
            document.getElementById("restartSound").play();
            startGame();
        }

        document.addEventListener("keydown", (e) => {
            if (e.code === "Space" && gameRunning) {
                bird.velocity = bird.lift;  /* Lift the bird upwards */
            }
        });

        document.addEventListener("touchstart", () => {
            if (gameRunning) {
                bird.velocity = bird.lift;  /* Lift the bird upwards on touch */
            }
        });

        initializeGame();
    </script>
</body>
</html>

# Flappybird-<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Flappy Bird</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: linear-gradient(to bottom, #56ccf2, #2f80ed);
      font-family: Arial, sans-serif;
      overflow: hidden;
    }

    #game-container {
      position: relative;
      width: 400px;
      height: 600px;
      overflow: hidden;
      background: linear-gradient(to bottom, #87CEEB, #1E90FF);
      border: 3px solid #2c3e50;
      border-radius: 10px;
      box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
    }

    #bird {
      position: absolute;
      width: 40px;
      height: 30px;
      background-image: url('https://iili.io/35M28kg.png');
      background-size: contain;
      background-repeat: no-repeat;
      z-index: 10;
      transition: transform 0.1s ease;
    }

    .pipe {
      position: absolute;
      width: 70px;
      background: linear-gradient(to right, #27ae60, #2ecc71);
      border: 3px solid #16a085;
      border-radius: 5px;
      box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.2);
      z-index: 5;
    }

    #ground {
      position: absolute;
      bottom: 0;
      width: 100%;
      height: 30px;
      background: linear-gradient(to right, #f39c12, #e67e22);
      z-index: 15;
      border-top: 3px solid #d35400;
    }

    #stats {
      position: absolute;
      top: 15px;
      left: 0;
      right: 0;
      display: flex;
      justify-content: center;
      gap: 40px;
      z-index: 20;
      background-color: rgba(0, 0, 0, 0.3);
      padding: 8px 0;
      backdrop-filter: blur(5px);
    }

    .stat {
      font-size: 16px;
      font-weight: bold;
      color: white;
      text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
      display: flex;
      align-items: center;
    }

    #game-over {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 25px;
      border-radius: 15px;
      text-align: center;
      display: none;
      z-index: 30;
      width: 80%;
      max-width: 300px;
      box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
    }

    #restart-btn {
      margin-top: 20px;
      padding: 12px 25px;
      background: linear-gradient(to right, #2ecc71, #27ae60);
      color: white;
      border: none;
      border-radius: 30px;
      cursor: pointer;
      font-size: 16px;
      font-weight: bold;
      transition: all 0.3s ease;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }

    #restart-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
      background: linear-gradient(to right, #27ae60, #2ecc71);
    }

    .cloud {
      position: absolute;
      background-color: rgba(255, 255, 255, 0.8);
      border-radius: 50%;
      z-index: 1;
    }
  </style>
</head>
<body>
  <div id="game-container">
    <div id="bird"></div>
    <div id="ground"></div>
    <div id="stats">
      <div class="stat">⏱️ <span id="distance">0</span>m</div>
      <div class="stat">🏆 <span id="score">0</span></div>
    </div>
    <div id="game-over">
      <h2 style="margin-bottom: 15px;">Game Over!</h2>
      <p style="font-size: 18px;">Distance: <span id="final-distance">0</span>m</p>
      <p style="font-size: 18px;">Score: <span id="final-score">0</span></p>
      <button id="restart-btn">Play Again</button>
    </div>
  </div>

  <script>
    // Game variables
    const bird = document.getElementById('bird');
    const gameContainer = document.getElementById('game-container');
    const scoreElement = document.getElementById('score');
    const distanceElement = document.getElementById('distance');
    const gameOverElement = document.getElementById('game-over');
    const finalScoreElement = document.getElementById('final-score');
    const finalDistanceElement = document.getElementById('final-distance');
    const restartBtn = document.getElementById('restart-btn');
    const ground = document.getElementById('ground');

    // Enhanced physics variables
    let birdPosition = 300;
    let birdVelocity = 0;
    let birdRotation = 0;
    let gravity = 0.6;
    let jumpForce = -9;
    let gameRunning = true;
    let score = 0;
    let distance = 0;
    let pipes = [];
    let clouds = [];
    let pipeGap = 180;
    let pipeFrequency = 2000;
    let lastPipeTime = 0;
    let distanceInterval;
    let gameStartTime;
    let gameSpeed = 2;
    let basePipeSpeed = 2;
    let difficultyInterval;
    let parallaxOffset = 0;

    // Set initial positions
    bird.style.top = birdPosition + 'px';
    bird.style.left = '100px';

    // Create initial clouds
    function createInitialClouds() {
      for (let i = 0; i < 5; i++) {
        createCloud(true);
      }
    }

    // Create a cloud
    function createCloud(initial = false) {
      const cloud = document.createElement('div');
      cloud.className = 'cloud';
      
      const size = Math.random() * 50 + 30;
      const posX = initial ? 
        Math.random() * gameContainer.clientWidth : 
        gameContainer.clientWidth + size;
      const posY = Math.random() * (gameContainer.clientHeight * 0.6);
      const opacity = Math.random() * 0.5 + 0.3;
      
      cloud.style.width = `${size}px`;
      cloud.style.height = `${size * 0.6}px`;
      cloud.style.left = `${posX}px`;
      cloud.style.top = `${posY}px`;
      cloud.style.opacity = opacity;
      
      gameContainer.appendChild(cloud);
      clouds.push({
        element: cloud,
        speed: Math.random() * 0.5 + 0.3
      });
    }

    // Update clouds
    function updateClouds() {
      for (let i = clouds.length - 1; i >= 0; i--) {
        const cloud = clouds[i];
        const currentLeft = parseFloat(cloud.element.style.left);
        const newLeft = currentLeft - cloud.speed;
        
        cloud.element.style.left = `${newLeft}px`;
        
        if (newLeft < -100) {
          gameContainer.removeChild(cloud.element);
          clouds.splice(i, 1);
          createCloud();
        }
      }
    }

    // Event listeners
    document.addEventListener('keydown', function(e) {
      if (e.code === 'Space') {
        jump();
      }
    });
    gameContainer.addEventListener('click', jump);
    restartBtn.addEventListener('click', restartGame);

    // Jump function with enhanced physics
    function jump() {
      if (!gameRunning) return;
      birdVelocity = jumpForce;
      birdRotation = -20;
      bird.style.transform = `rotate(${birdRotation}deg)`;
    }

    // Update bird rotation based on velocity
    function updateBirdRotation() {
      birdRotation = Math.min(Math.max(birdVelocity * 3, -20), 90);
      bird.style.transform = `rotate(${birdRotation}deg)`;
    }

    // Increase difficulty over time
    function increaseDifficulty() {
      gameSpeed += 0.05;
      pipeFrequency = Math.max(1000, pipeFrequency - 20);
    }

    // Game loop with enhanced physics
    function gameLoop(timestamp) {
      if (!gameRunning) return;

      // Update bird physics
      birdVelocity += gravity;
      birdPosition += birdVelocity;
      bird.style.top = birdPosition + 'px';
      updateBirdRotation();

      // Check for collisions with ground and ceiling
      if (birdPosition < 0 || birdPosition > gameContainer.clientHeight - ground.clientHeight - bird.clientHeight) {
        gameOver();
        return;
      }

      // Generate pipes
      if (timestamp - lastPipeTime > pipeFrequency) {
        createPipe();
        lastPipeTime = timestamp;
      }

      // Move pipes and check for collisions
      for (let i = pipes.length - 1; i >= 0; i--) {
        const pipe = pipes[i];
        const pipeLeft = parseFloat(pipe.style.left);

        // Move pipe with game speed
        pipe.style.left = `${pipeLeft - (basePipeSpeed * gameSpeed)}px`;

        // Remove off-screen pipes
        if (pipeLeft < -70) {
          gameContainer.removeChild(pipe);
          pipes.splice(i, 1);
          continue;
        }

        // Check for collision with bird
        if (checkCollision(bird, pipe)) {
          gameOver();
          return;
        }

        // Score points when passing pipes
        if (pipe.dataset.passed !== 'true' && pipeLeft + 70 < 100) {
          pipe.dataset.passed = 'true';
          score++;
          scoreElement.textContent = score;
        }
      }

      // Update clouds
      updateClouds();

      // Update parallax effect
      parallaxOffset += 0.5;
      ground.style.backgroundPositionX = `-${parallaxOffset % 60}px`;

      requestAnimationFrame(gameLoop);
    }

    // Create a new pipe with realistic variations
    function createPipe() {
      const minHeight = 80;
      const maxHeight = gameContainer.clientHeight - pipeGap - minHeight - ground.clientHeight;
      const topHeight = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;

      // Top pipe
      const topPipe = document.createElement('div');
      topPipe.className = 'pipe';
      topPipe.style.left = `${gameContainer.clientWidth}px`;
      topPipe.style.top = '0';
      topPipe.style.height = `${topHeight}px`;
      topPipe.style.borderBottomLeftRadius = '0';
      topPipe.style.borderBottomRightRadius = '0';
      gameContainer.appendChild(topPipe);
      pipes.push(topPipe);

      // Bottom pipe
      const bottomPipe = document.createElement('div');
      bottomPipe.className = 'pipe';
      bottomPipe.style.left = `${gameContainer.clientWidth}px`;
      bottomPipe.style.top = `${topHeight + pipeGap}px`;
      bottomPipe.style.height = `${gameContainer.clientHeight - topHeight - pipeGap - ground.clientHeight}px`;
      bottomPipe.style.borderTopLeftRadius = '0';
      bottomPipe.style.borderTopRightRadius = '0';
      gameContainer.appendChild(bottomPipe);
      pipes.push(bottomPipe);
    }

    // Realistic collision detection
    function checkCollision(bird, pipe) {
      const birdRect = bird.getBoundingClientRect();
      const pipeRect = pipe.getBoundingClientRect();
      const containerRect = gameContainer.getBoundingClientRect();

      // Adjust positions relative to game container
      const birdLeft = birdRect.left - containerRect.left;
      const birdRight = birdRect.right - containerRect.left;
      const birdTop = birdRect.top - containerRect.top;
      const birdBottom = birdRect.bottom - containerRect.top;

      const pipeLeft = pipeRect.left - containerRect.left;
      const pipeRight = pipeRect.right - containerRect.left;
      const pipeTop = pipeRect.top - containerRect.top;
      const pipeBottom = pipeRect.bottom - containerRect.top;

      // More precise collision detection
      return (
        birdRight > pipeLeft + 5 &&
        birdLeft < pipeRight - 5 &&
        birdBottom > pipeTop + 5 &&
        birdTop < pipeBottom - 5
      );
    }

    // Update distance
    function updateDistance() {
      const secondsElapsed = (Date.now() - gameStartTime) / 1000;
      distance = Math.floor(secondsElapsed * 10); // 10m per second
      distanceElement.textContent = distance;
    }

    // Game over function
    function gameOver() {
      gameRunning = false;
      clearInterval(distanceInterval);
      clearInterval(difficultyInterval);
      finalScoreElement.textContent = score;
      finalDistanceElement.textContent = distance;
      gameOverElement.style.display = 'block';
      
      // Visual feedback
      bird.style.transform = 'rotate(90deg)';
      gameContainer.style.backgroundColor = '#ff6b6b';
      setTimeout(() => {
        gameContainer.style.backgroundColor = '';
      }, 200);
    }

    // Restart game function
    function restartGame() {
      // Reset bird
      birdPosition = 300;
      birdVelocity = 0;
      birdRotation = 0;
      bird.style.top = birdPosition + 'px';
      bird.style.transform = 'rotate(0deg)';
      bird.style.opacity = '1';

      // Remove all pipes
      pipes.forEach(pipe => gameContainer.removeChild(pipe));
      pipes = [];

      // Remove all clouds
      clouds.forEach(cloud => gameContainer.removeChild(cloud.element));
      clouds = [];

      // Reset stats
      score = 0;
      distance = 0;
      gameSpeed = 2;
      pipeFrequency = 2000;
      scoreElement.textContent = score;
      distanceElement.textContent = distance;

      // Hide game over screen
      gameOverElement.style.display = 'none';

      // Start game elements
      createInitialClouds();

      // Start game
      gameRunning = true;
      gameStartTime = Date.now();
      distanceInterval = setInterval(updateDistance, 100);
      difficultyInterval = setInterval(increaseDifficulty, 3000);
      lastPipeTime = performance.now();
      requestAnimationFrame(gameLoop);
    }

    // Initialize game
    createInitialClouds();
    restartGame();
  </script>
</body>
</html>

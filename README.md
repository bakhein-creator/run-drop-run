<!DOCTYPE html>
<html lang="ko">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>강릉: 물방울 질주</title>
   <script src="https://cdn.tailwindcss.com"></script>
   <style>
       @import url('https://fonts.googleapis.com/css2?family=Jua&display=swap');
       body {
           font-family: 'Jua', sans-serif;
           background-color: #f0f9ff;
           color: #1e40af;
           display: flex;
           flex-direction: column;
           justify-content: center;
           align-items: center;
           min-height: 100vh;
           padding: 1rem;
           overflow: hidden;
       }
       .game-container {
           background-color: #ffffff;
           border-radius: 1.5rem;
           box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
           overflow: hidden;
           display: flex;
           flex-direction: column;
           align-items: center;
           width: 100%;
           max-width: 900px;
           padding: 2rem;
           position: relative;
       }
       .header-container {
           width: 100%;
           display: flex;
           flex-direction: column;
           align-items: center;
       }
       .title-and-score {
           display: flex;
           justify-content: center;
           align-items: center;
           gap: 1.5rem;
           margin-bottom: 1rem;
       }
       .score-info {
           display: flex;
           flex-direction: column;
           text-align: right;
           font-weight: bold;
           font-size: 1rem;
       }
       .info-label {
           color: #4b5563;
           font-size: 0.875rem;
       }
       .info-value {
           color: #1e40af;
           font-size: 1.125rem;
       }
       canvas {
           border-radius: 1rem;
           border: 4px solid #1e40af;
           touch-action: none;
           width: 100%;
           aspect-ratio: 16 / 9;
       }
       .game-ui {
           background-color: #e0f2fe;
           border-radius: 1rem;
           padding: 1rem;
           width: 100%;
           text-align: center;
           margin-top: 1rem;
           display: flex;
           justify-content: space-between;
           align-items: center;
           font-weight: bold;
       }
       .modal {
           position: fixed;
           top: 0;
           left: 0;
           width: 100%;
           height: 100%;
           background-color: rgba(0, 0, 0, 0.6);
           display: flex;
           justify-content: center;
           align-items: center;
           z-index: 1000;
       }
       .modal-content {
           background-color: white;
           padding: 2rem;
           border-radius: 1rem;
           box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
           max-width: 400px;
           width: 90%;
           text-align: center;
           display: flex;
           flex-direction: column;
           align-items: center;
       }
       .modal-button {
           background-color: #3b82f6;
           color: white;
           padding: 0.75rem 1.5rem;
           border-radius: 9999px;
           margin-top: 1rem;
       }
       .modal-button-red {
           background-color: #ef4444;
       }
       .hidden {
           display: none;
       }
       .spinner {
           border: 4px solid rgba(0, 0, 0, 0.1);
           width: 36px;
           height: 36px;
           border-radius: 50%;
           border-left-color: #3b82f6;
           animation: spin 1s ease infinite;
           margin-top: 1rem;
       }
       @keyframes spin {
           0% { transform: rotate(0deg); }
           100% { transform: rotate(360deg); }
       }
       #resultImage {
           margin-top: 1.5rem;
           border-radius: 0.5rem;
           box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
       }
   </style>
</head>
<body>


<div class="game-container">
   <div class="header-container">
       <div class="title-and-score">
           <h1 class="text-4xl font-bold text-blue-800">💧 강릉: 물방울 질주 🏃</h1>
           <div class="score-info">
               <div><span class="info-label">현재 점수:</span> <span id="score" class="info-value">0</span></div>
               <div><span class="info-label">최고 점수:</span> <span id="high-score" class="info-value">0</span></div>
           </div>
       </div>
       <p class="text-lg text-center text-blue-600 mb-6">마른 땅을 피해 물방울을 모으세요!</p>
   </div>


   <canvas id="gameCanvas"></canvas>


   <div class="game-ui">
       <div><span class="info-label">남은 물:</span> <span id="water-level" class="info-value">100%</span></div>
       <div><span class="info-label">보석:</span> <span id="gems" class="info-value">0</span></div>
   </div>
</div>


<div id="startModal" class="modal">
   <div class="modal-content">
       <p class="text-xl font-bold mb-4">강릉: 물방울 질주</p>
       <p class="mb-4">메마른 땅을 가로질러 물방울을 모으고, 장애물을 피하세요!</p>
       <p class="mb-4">마우스 클릭 또는 스페이스바를 누르면 더블 점프합니다. 파란색 물방울은 1점, 노란색은 5점, 빨간색은 10점입니다! 보라색 보석은 부활에 사용됩니다.</p>
       <button id="startButton" class="modal-button">게임 시작</button>
   </div>
</div>


<div id="reviveModal" class="modal hidden">
   <div class="modal-content">
       <p class="text-xl font-bold mb-4">위험해요! 보석을 사용하시겠어요?</p>
       <p id="reviveMessage" class="mb-4">보석 1개를 사용하여 다시 시작할 수 있습니다. 현재 보석: <span id="reviveGemCount" class="font-bold">0</span></p>
       <div class="flex gap-4">
           <button id="reviveButton" class="modal-button">부활</button>
           <button id="endGameButton" class="modal-button modal-button-red">게임 종료</button>
       </div>
   </div>
</div>


<div id="endModal" class="modal hidden">
   <div class="modal-content">
       <p class="text-xl font-bold mb-4">게임 오버!</p>
       <p id="final-score" class="text-3xl font-bold text-blue-800 mb-4"></p>
       <p id="end-message" class="text-gray-700 text-sm mt-2"></p>
       <div id="loading" class="spinner"></div>
       <p id="geminiMessage" class="text-gray-700 text-sm mt-2"></p>
       <img id="resultImage" class="mt-4" style="display:none;" />
       <button id="restartButton" class="modal-button">다시 시작</button>
   </div>
</div>


<audio id="background-music" loop>
 <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" type="audio/mp3">
 Your browser does not support the audio element.
</audio>


<script>
   const canvas = document.getElementById('gameCanvas');
   const ctx = canvas.getContext('2d');
   const backgroundMusic = document.getElementById('background-music');


   const waterLevelEl = document.getElementById('water-level');
   const scoreEl = document.getElementById('score');
   const highScoreEl = document.getElementById('high-score');
   const gemsEl = document.getElementById('gems');
   const startModal = document.getElementById('startModal');
   const startButton = document.getElementById('startButton');
   const endModal = document.getElementById('endModal');
   const restartButton = document.getElementById('restartButton');
   const finalScoreEl = document.getElementById('final-score');
   const endMessageEl = document.getElementById('end-message');
   const reviveModal = document.getElementById('reviveModal');
   const reviveButton = document.getElementById('reviveButton');
   const endGameButton = document.getElementById('endGameButton');
   const reviveGemCountEl = document.getElementById('reviveGemCount');
   const loadingSpinner = document.getElementById('loading');
   const geminiMessageEl = document.getElementById('geminiMessage');
   const resultImageEl = document.getElementById('resultImage');


   // Game state
   let gameLoopId;
   let isGameRunning = false;
   let score = 0;
   let waterLevel = 100;
   let gems = 0;


   // Player
   const player = {
       x: 50,
       y: 0,
       width: 30, // Player's circle diameter
       radius: 15,
       velocityY: 0,
       jumpCount: 0,
       isInvincible: false,
       invincibleTimer: null
   };


   // Game physics
   const gravity = 0.5;
   const jumpPower = -12;
   let groundLevel;


   // Obstacles and collectibles
   const obstacles = [];
   const collectibles = [];
   // 게임 시작 시 초기 속도 설정 (조금 더 빠르게 시작)
   const initialObstacleSpeed = 2.5;
   let obstacleSpeed = initialObstacleSpeed;
   let obstacleSpawnRate = 120;
   let obstacleSpawnCounter = 0;


   // Helper functions
   function showModal(id) {
       document.getElementById(id).classList.remove('hidden');
   }


   function hideModal(id) {
       document.getElementById(id).classList.add('hidden');
   }


   // New, more accurate collision detection for circle and rectangle
   function checkCircleRectCollision(circle, rect) {
       const distX = Math.abs(circle.x - rect.x - rect.width / 2);
       const distY = Math.abs(circle.y - rect.y - rect.height / 2);


       if (distX > (rect.width / 2 + circle.radius)) { return false; }
       if (distY > (rect.height / 2 + circle.radius)) { return false; }


       if (distX <= (rect.width / 2)) { return true; }
       if (distY <= (rect.height / 2)) { return true; }


       const dx = distX - rect.width / 2;
       const dy = distY - rect.height / 2;
       return (dx * dx + dy * dy <= (circle.radius * circle.radius));
   }


   // Drawing functions
   function drawBackground() {
       // Sky
       ctx.fillStyle = '#6cb2e6'; // A slightly darker, more distinct blue
       ctx.fillRect(0, 0, canvas.width, groundLevel);


       // Distant hills
       ctx.fillStyle = '#4CAF50';
       ctx.beginPath();
       ctx.moveTo(0, groundLevel);
       ctx.lineTo(canvas.width / 4, groundLevel - 50);
       ctx.lineTo(canvas.width / 2, groundLevel - 20);
       ctx.lineTo(canvas.width * 0.75, groundLevel - 60);
       ctx.lineTo(canvas.width, groundLevel - 30);
       ctx.lineTo(canvas.width, groundLevel);
       ctx.closePath();
       ctx.fill();


       // Foreground ground
       ctx.fillStyle = '#a3e635';
       ctx.fillRect(0, groundLevel, canvas.width, canvas.height - groundLevel);
   }


   function drawPlayer() {
       ctx.fillStyle = player.isInvincible ? 'rgba(59, 130, 246, 0.5)' : '#3b82f6';
       ctx.beginPath();
       ctx.arc(player.x + player.radius, player.y + player.radius, player.radius, 0, Math.PI * 2);
       ctx.fill();
      
       if (player.isInvincible) {
           ctx.strokeStyle = 'rgba(59, 130, 246, 0.8)';
           ctx.lineWidth = 3;
           ctx.beginPath();
           ctx.arc(player.x + player.radius, player.y + player.radius, player.radius + 5 + Math.sin(Date.now() * 0.01) * 3, 0, Math.PI * 2);
           ctx.stroke();
       }
   }


   function drawObstacles() {
       ctx.fillStyle = '#8b4513';
       obstacles.forEach(obs => {
           ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
       });
   }


   function drawCollectibles() {
       collectibles.forEach(col => {
           if (col.type === 'waterdrop') {
               ctx.fillStyle = '#3498db'; // Brighter blue for visibility
               ctx.beginPath();
               ctx.arc(col.x, col.y, col.radius, 0, Math.PI * 2);
               ctx.fill();
           } else if (col.type === 'super_waterdrop') {
               ctx.fillStyle = '#FFD700'; // Gold color for super waterdrop
               ctx.beginPath();
               ctx.arc(col.x, col.y, col.radius, 0, Math.PI * 2);
               ctx.fill();
           } else if (col.type === 'bonus_waterdrop') {
               ctx.fillStyle = '#dc2626'; // Red color for bonus waterdrop
               ctx.beginPath();
               ctx.arc(col.x, col.y, col.radius + 2, 0, Math.PI * 2);
               ctx.fill();
           } else if (col.type === 'shield') {
               ctx.fillStyle = '#8b5cf6'; // Purple for shield
               ctx.beginPath();
               ctx.rect(col.x - col.radius, col.y - col.radius, col.radius * 2, col.radius * 2);
               ctx.fill();
           } else if (col.type === 'gem') {
               ctx.fillStyle = '#9b59b6'; // 보석 색상 (진한 보라색)
               ctx.beginPath();
               // 다이아몬드 형태를 그립니다
               ctx.moveTo(col.x, col.y - col.radius);
               ctx.lineTo(col.x + col.radius, col.y);
               ctx.lineTo(col.x, col.y + col.radius);
               ctx.lineTo(col.x - col.radius, col.y);
               ctx.closePath();
               ctx.fill();
           }
          
           // Sparkling effect for rare items
           if (col.type === 'bonus_waterdrop' || col.type === 'shield' || col.type === 'gem') {
               ctx.save();
               ctx.beginPath();
               const sparkleRadius = col.radius + 5 + Math.sin(Date.now() * 0.01) * 3;
               const sparkleColor = col.type === 'bonus_waterdrop' ? '#FFD700' : (col.type === 'shield' ? '#8b5cf6' : '#c0392b');
               const opacity = Math.abs(Math.sin(Date.now() * 0.005));
               ctx.strokeStyle = `rgba(${parseInt(sparkleColor.slice(1,3), 16)}, ${parseInt(sparkleColor.slice(3,5), 16)}, ${parseInt(sparkleColor.slice(5,7), 16)}, ${opacity})`;
               ctx.lineWidth = 2;
               ctx.arc(col.x, col.y, sparkleRadius, 0, Math.PI * 2);
               ctx.stroke();
               ctx.restore();
           }
       });
   }


   function draw() {
       ctx.clearRect(0, 0, canvas.width, canvas.height);
      
       drawBackground();
       drawPlayer();
       drawObstacles();
       drawCollectibles();
   }


   function update() {
       player.velocityY += gravity;
       player.y += player.velocityY;


       // Player collision with ground
       if (player.y >= groundLevel - player.width) {
           player.y = groundLevel - player.width;
           player.velocityY = 0;
           player.jumpCount = 0;
       }


       obstacleSpawnCounter++;
       if (obstacleSpawnCounter >= obstacleSpawnRate) {
           // 장애물 길이를 조금 더 길게 조정
           const width = 50 + Math.random() * 80;
           const height = 30 + Math.random() * 80;
           const obstacle = {
               x: canvas.width,
               y: groundLevel - height,
               width: width,
               height: height
           };
           obstacles.push(obstacle);
           obstacleSpawnCounter = 0;
       }
      
       // Collectible generation probabilities
       const rand = Math.random();
       if (rand < 0.015) { // Blue waterdrop (1 point)
           const collectible = {
               x: canvas.width,
               y: groundLevel - 100 - Math.random() * 80,
               radius: 12, // Increased size
               type: 'waterdrop'
           };
           collectibles.push(collectible);
       } else if (rand < 0.015 + 0.006) { // Yellow waterdrop (5 points)
            const superWaterdrop = {
               x: canvas.width,
               y: groundLevel - 100 - Math.random() * 80,
               radius: 13, // Decreased size
               type: 'super_waterdrop'
           };
           collectibles.push(superWaterdrop);
       } else if (rand < 0.015 + 0.006 + 0.001) { // Red waterdrop (10 points) - Made less frequent
            const bonusWaterdrop = {
               x: canvas.width,
               y: groundLevel - 100 - Math.random() * 80,
               radius: 15, // Decreased size
               type: 'bonus_waterdrop'
           };
           collectibles.push(bonusWaterdrop);
       } else if (rand < 0.015 + 0.006 + 0.001 + 0.001) { // Shield (purple)
            const shield = {
               x: canvas.width,
               y: groundLevel - 100 - Math.random() * 80,
               radius: 10, // Decreased size
               type: 'shield'
           };
           collectibles.push(shield);
       } else if (rand < 0.015 + 0.006 + 0.001 + 0.001 + 0.003) { // Gem (purple)
            const gem = {
               x: canvas.width,
               y: groundLevel - 100 - Math.random() * 80,
               radius: 10,
               type: 'gem'
           };
           collectibles.push(gem);
       }


       // 시간이 지남에 따라 점진적으로 속도 증가
       obstacleSpeed += 0.0004;
      
       obstacles.forEach(obs => obs.x -= obstacleSpeed);
       collectibles.forEach(col => col.x -= obstacleSpeed);
      
       // Remove off-screen obstacles and collectibles
       if (obstacles.length > 0 && obstacles[0].x < -obstacles[0].width) {
           obstacles.shift();
           score++;
           waterLevel -= 0.1;
       }


       if (collectibles.length > 0 && collectibles[0].x < -collectibles[0].radius) {
           collectibles.shift();
       }


       // Check for collision with obstacles
       const playerCircle = { x: player.x + player.radius, y: player.y + player.radius, radius: player.radius };
       obstacles.forEach(obs => {
           const isColliding = checkCircleRectCollision(playerCircle, obs);
           if (isColliding) {
               // 플레이어가 장애물 위에 착지했는지 더 안정적으로 확인
               const onTop = player.velocityY >= 0 && (player.y + player.width - 5) <= obs.y;
               if (onTop) {
                   player.y = obs.y - player.width;
                   player.velocityY = 0;
                   player.jumpCount = 0;
               } else {
                   // 측면 또는 하단 충돌
                   if (!player.isInvincible) {
                       isGameRunning = false;
                   }
               }
           }
       });
      
       // Check for collision with collectibles
       collectibles.forEach((col, index) => {
           const dx = player.x + player.radius - col.x;
           const dy = player.y + player.radius - col.y;
           const distance = Math.sqrt(dx * dx + dy * dy);


           if (distance < player.radius + col.radius) {
               if (col.type === 'waterdrop') {
                   waterLevel = Math.min(100, waterLevel + 5);
                   score += 1; // 1점
               } else if (col.type === 'super_waterdrop') {
                   waterLevel = Math.min(100, waterLevel + 15);
                   score += 5; // 5점
               } else if (col.type === 'bonus_waterdrop') {
                   waterLevel = Math.min(100, waterLevel + 20);
                   score += 10; // 10점
               } else if (col.type === 'shield') {
                   player.isInvincible = true;
                   if (player.invincibleTimer) {
                       clearTimeout(player.invincibleTimer);
                   }
                   player.invincibleTimer = setTimeout(() => {
                       player.isInvincible = false;
                   }, 5000);
               } else if (col.type === 'gem') {
                   gems += 1;
               }
               collectibles.splice(index, 1);
           }
       });


       // 점수에 따라 장애물 생성 속도 증가
       if (Math.floor(score) % 50 === 0 && Math.floor(score) > 0) {
           obstacleSpeed += 0.05;
           obstacleSpawnRate = Math.max(60, obstacleSpawnRate - 1);
       }
      
       waterLevel = Math.max(0, waterLevel);
       if (waterLevel <= 0) {
           isGameRunning = false;
       }


       waterLevelEl.textContent = `${Math.floor(waterLevel)}%`;
       scoreEl.textContent = Math.floor(score);
       highScoreEl.textContent = localStorage.getItem('highScore') || 0;
       gemsEl.textContent = gems;
   }


   function gameLoop() {
       if (!isGameRunning) {
           checkRevive();
           return;
       }
       update();
       draw();
       gameLoopId = requestAnimationFrame(gameLoop);
   }
  
   function checkRevive() {
       if (gems > 0) {
           showReviveModal();
       } else {
           finishGame();
       }
   }


   function showReviveModal() {
       backgroundMusic.pause();
       reviveGemCountEl.textContent = gems;
       showModal('reviveModal');
   }


   function revive() {
       gems -= 1;
       waterLevel = 50; // Revive with 50% water
       player.velocityY = 0;
       player.jumpCount = 0;
       isGameRunning = true;
       hideModal('reviveModal');
       backgroundMusic.play();
       gameLoop();
   }


   function finishGame() {
       // 배경 음악 정지
       backgroundMusic.pause();
       backgroundMusic.currentTime = 0;


       finalScoreEl.textContent = `최종 점수: ${Math.floor(score)}`;
       endMessageEl.textContent = waterLevel <= 0 ? "물이 다 말랐습니다..." : "장애물에 부딪혔습니다...";
      
       // Save high score
       const savedHighScore = localStorage.getItem('highScore') || 0;
       if (score > savedHighScore) {
           localStorage.setItem('highScore', Math.floor(score));
           highScoreEl.textContent = Math.floor(score);
       }
      
       showModal('endModal');
       getGeminiResult(Math.floor(score));
   }


   document.addEventListener('keydown', (e) => {
       if (e.code === 'Space' && player.jumpCount < 2) {
           player.velocityY = jumpPower;
           player.jumpCount++;
       }
   });


   canvas.addEventListener('click', (e) => {
       if (player.jumpCount < 2) {
           player.velocityY = jumpPower;
           player.jumpCount++;
       }
   });


   // Gemini API integration
   async function getGeminiResult(finalScore) {
       const apiKey = "";
       const apiUrl = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=" + apiKey;
       const imageApiUrl = "https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=" + apiKey;


       let messagePrompt;
       let imagePrompt;


       if (finalScore >= 100) {
           messagePrompt = `사용자의 게임 점수는 ${finalScore}점입니다. "강릉: 물방울 질주"라는 게임에서 고득점을 기록했습니다. 사용자를 칭찬하고 다음 플레이를 격려하는 창의적인 한국어 게임 오버 메시지를 짧게 작성해주세요. 물방울, 바다, 강릉 같은 키워드를 사용해 주세요.`;
           imagePrompt = `A cheerful, cute waterdrop character with a gold medal, standing on a sunny beach in Gangneung. The style is 3D animation, bright and colorful.`;
       } else if (finalScore >= 30) {
           messagePrompt = `사용자의 게임 점수는 ${finalScore}점입니다. "강릉: 물방울 질주"라는 게임에서 괜찮은 점수를 기록했습니다. 더 열심히 하면 더 잘할 수 있다고 격려하는 창의적인 한국어 게임 오버 메시지를 짧게 작성해주세요.`;
           imagePrompt = `A determined waterdrop character standing in front of a blue ocean and a sunny sky, getting ready to run. The style is cute and stylized.`;
       } else {
           messagePrompt = `사용자의 게임 점수는 ${finalScore}점입니다. "강릉: 물방울 질주"라는 게임에서 낮은 점수를 기록했습니다. 다음에는 더 잘할 수 있다고 격려하고, 재도전을 유도하는 창의적인 한국어 게임 오버 메시지를 짧게 작성해주세요.`;
           imagePrompt = `A sad waterdrop character looking at a cracked, dry ground with a desolate background. The style is simple and stylized.`;
       }


       loadingSpinner.style.display = 'block';
       geminiMessageEl.textContent = '';
       resultImageEl.style.display = 'none';


       try {
           // Text generation
           const messagePayload = { contents: [{ parts: [{ text: messagePrompt }] }] };
           const messageResponse = await fetch(apiUrl, {
               method: 'POST',
               headers: { 'Content-Type': 'application/json' },
               body: JSON.stringify(messagePayload)
           });
           const messageResult = await messageResponse.json();
           const generatedMessage = messageResult?.candidates?.[0]?.content?.parts?.[0]?.text || "메시지 생성에 실패했습니다.";
           geminiMessageEl.textContent = `✨ ${generatedMessage} ✨`;


           // Image generation
           const imagePayload = { instances: { prompt: imagePrompt }, parameters: { sampleCount: 1 } };
           const imageResponse = await fetch(imageApiUrl, {
               method: 'POST',
               headers: { 'Content-Type': 'application/json' },
               body: JSON.stringify(imagePayload)
           });
           const imageResult = await imageResponse.json();
           const base64Data = imageResult?.predictions?.[0]?.bytesBase64Encoded;
           if (base64Data) {
               resultImageEl.src = `data:image/png;base64,${base64Data}`;
               resultImageEl.style.display = 'block';
           } else {
               console.error("이미지 생성에 실패했습니다.");
           }


       } catch (error) {
           console.error("API 호출 중 오류가 발생했습니다:", error);
           geminiMessageEl.textContent = "결과 생성 중 오류가 발생했습니다.";
       } finally {
           loadingSpinner.style.display = 'none';
       }
   }


   function startGame() {
       // 배경 음악 재생
       backgroundMusic.play();


       // Load high score
       const savedHighScore = localStorage.getItem('highScore') || 0;
       highScoreEl.textContent = savedHighScore;


       canvas.width = canvas.parentElement.clientWidth;
       canvas.height = canvas.width / (16/9);
       groundLevel = canvas.height - 50;


       isGameRunning = true;
       score = 0;
       waterLevel = 100;
       player.y = groundLevel - player.width;
       player.velocityY = 0;
       player.jumpCount = 0;
       obstacles.length = 0;
       collectibles.length = 0;
       // 게임 시작 시 속도 재설정
       obstacleSpeed = initialObstacleSpeed;
       obstacleSpawnRate = 120;
       obstacleSpawnCounter = 0;


       hideModal('startModal');
       hideModal('endModal');
       hideModal('reviveModal');
       gameLoop();
   }
  
   startButton.addEventListener('click', startGame);
   restartButton.addEventListener('click', startGame);
   reviveButton.addEventListener('click', revive);
   endGameButton.addEventListener('click', finishGame);


   window.onload = function() {
       showModal('startModal');
       canvas.width = canvas.parentElement.clientWidth;
       canvas.height = canvas.width / (16/9);
       groundLevel = canvas.height - 50;
       player.y = groundLevel - player.width;


       const savedHighScore = localStorage.getItem('highScore') || 0;
       highScoreEl.textContent = savedHighScore;
   }


</script>
</body>
</html>




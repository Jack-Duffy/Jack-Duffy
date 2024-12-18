---
layout: base
title: backup
permalink: /backup/
---


---


<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Snake Game</title>
  <style>
    body {
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      font-family: Arial, sans-serif;
      background-color: #f0f0f0;
    }
    canvas {
      border: 10px solid darkgreen;
      background: repeating-linear-gradient(
          0deg,
          lightgreen 20px,
          green 20px,
          green 40px
        ),
        repeating-linear-gradient(
          90deg,
          lightgreen 20px,
          green 20px,
          green 40px
        );
    }
    button {
      margin: 10px;
      font-size: 16px;
      background: none;
      border: none;
      color: #0047ab;
      text-decoration: underline;
      cursor: pointer;
    }
    button:hover {
      text-decoration: none;
    }
    #setting, #gameover {
      display: none;
    }
    #score_display {
      font-size: 18px;
      margin: 10px;
      color: #333;
    }
  </style>
</head>
<body>
  <div id="menu">
    <button id="new_game">New Game</button>
    <button id="setting_menu">Settings</button>
  </div>
  <div id="setting">
    <h3>Settings</h3>
    <label>
      Wall Collisions:
      <input type="radio" name="wall" value="true" checked> On
      <input type="radio" name="wall" value="false"> Off
    </label>
    <br>
    <label>
      Speed:
      <input type="radio" name="speed" value="200" checked> Easy
      <input type="radio" name="speed" value="100"> Medium
      <input type="radio" name="speed" value="50"> Hard
    </label>
    <br>
    <button id="new_game1">Start Game</button>
    <button id="back_to_menu">Back to Menu</button>
  </div>
  <div id="gameover">
    <h3>Game Over</h3>
    <p>Score: <span id="score_value">0</span></p>
    <p>High Score: <span id="high_score_value">0</span></p>
    <button id="new_game2">Play Again</button>
    <button id="setting_menu1">Settings</button>
  </div>
  <div id="score_display">Score: <span id="current_score">0</span></div>
  <canvas id="snake" width="400" height="400"></canvas>
</body>

<script>
/* Game Attributes */
/////////////////////////////////////////////////////////////
// Canvas & Context
const canvas = document.getElementById("snake");
const ctx = canvas.getContext("2d");

// HTML Elements
const SCREEN_MENU = -1, SCREEN_SNAKE = 0, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2;
const menuScreen = document.getElementById("menu");
const settingScreen = document.getElementById("setting");
const gameOverScreen = document.getElementById("gameover");
const currentScoreDisplay = document.getElementById("current_score");
const scoreDisplay = document.getElementById("score_value");
const highScoreDisplay = document.getElementById("high_score_value");

// Buttons
const startButtons = [document.getElementById("new_game"), document.getElementById("new_game1"), document.getElementById("new_game2")];
const settingsButtons = [document.getElementById("setting_menu"), document.getElementById("setting_menu1")];
const backButton = document.getElementById("back_to_menu");

// Game Settings
let wallCollision = true;
let speed = 200;

// Game State
let snake = [];
let direction = { x: 0, y: 0 };
let nextDirection = { x: 0, y: 0 };
let food = { x: 0, y: 0, color: "red" };
let score = 0;
let highScore = 0;
let frameInterval = 16;
let gameSpeed = 10; // Frames between updates
let frameCount = 0;

// Utility Functions
function showScreen(screen) {
  menuScreen.style.display = screen === SCREEN_MENU ? "block" : "none";
  settingScreen.style.display = screen === SCREEN_SETTING ? "block" : "none";
  gameOverScreen.style.display = screen === SCREEN_GAME_OVER ? "block" : "none";
  canvas.style.display = screen === SCREEN_SNAKE ? "block" : "none";
  document.getElementById("score_display").style.display = screen === SCREEN_SNAKE ? "block" : "none";
}

function generateFood() {
  const colors = ["red", "orange", "yellow"];
  let isOnSnake;
  do {
    isOnSnake = false;
    food.x = Math.floor(Math.random() * (canvas.width / 20)) * 20;
    food.y = Math.floor(Math.random() * (canvas.height / 20)) * 20;
    isOnSnake = snake.some(segment => segment.x === food.x && segment.y === food.y);
  } while (isOnSnake);
  food.color = colors[Math.floor(Math.random() * colors.length)];
}

/* Initialize Game */
function initializeGame() {
  snake = [{ x: 200, y: 200 }];
  direction = { x: 0, y: 0 };
  nextDirection = { x: 0, y: 0 };
  score = 0;
  currentScoreDisplay.textContent = score;
  frameCount = 0;
  generateFood();
  showScreen(SCREEN_SNAKE);
  requestAnimationFrame(gameLoop);
}

/* Game Loop */
function gameLoop() {
  frameCount++;
  if (frameCount >= gameSpeed) {
    frameCount = 0;

    // Update direction
    if ((nextDirection.x !== -direction.x || nextDirection.y !== -direction.y)) {
      direction = nextDirection;
    }

    // Move the snake
    const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };
    snake.unshift(head);

    // Check food collision
    if (head.x === food.x && head.y === food.y) {
      score++;
      currentScoreDisplay.textContent = score;
      generateFood();
    } else {
      snake.pop();
    }

    // Check wall collisions
    if (
      wallCollision &&
      (head.x < 0 || head.y < 0 || head.x >= canvas.width || head.y >= canvas.height)
    ) {
      endGame();
      return;
    }

    // Check self-collision
    if (snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)) {
      endGame();
      return;
    }
  }

  // Render game
  renderGame();
  requestAnimationFrame(gameLoop);
}

/* Render Game */
function renderGame() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Draw food
  ctx.fillStyle = food.color;
  ctx.fillRect(food.x, food.y, 20, 20);

  // Draw snake
  snake.forEach((segment, index) => {
    ctx.fillStyle = "blue";
    ctx.fillRect(segment.x, segment.y, 20, 20);

    if (index === 0) {
      // Draw eyes on the head
      ctx.fillStyle = "white";
      ctx.fillRect(segment.x + 5, segment.y + 5, 5, 5);
      ctx.fillRect(segment.x + 10, segment.y + 5, 5, 5);
    }
  });
}

/* End Game */
function endGame() {
  if (score > highScore) highScore = score;
  scoreDisplay.textContent = score;
  highScoreDisplay.textContent = highScore;
  showScreen(SCREEN_GAME_OVER);
}

/* Input Handling */
document.addEventListener("keydown", e => {
  if (e.key === "ArrowUp" || e.key === "w") nextDirection = { x: 0, y: -20 };
  if (e.key === "ArrowDown" || e.key === "s") nextDirection = { x: 0, y: 20 };
  if (e.key === "ArrowLeft" || e.key === "a") nextDirection = { x: -20, y: 0 };
  if (e.key === "ArrowRight" || e.key === "d") nextDirection = { x: 20, y: 0 };
});

/* Button Events */
startButtons.forEach(button => button.addEventListener("click", initializeGame));
settingsButtons.forEach(button => button.addEventListener("click", () => showScreen(SCREEN_SETTING)));
backButton.addEventListener("click", () => showScreen(SCREEN_MENU));
document.getElementsByName("wall").forEach(input =>
  input.addEventListener("change", () => (wallCollision = input.value === "true"))
);
document.getElementsByName("speed").forEach(input =>
  input.addEventListener("change", () => (gameSpeed = 200 / input.value))
);
</script>

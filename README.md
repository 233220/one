const canvas = document.getElementById('pong');
const ctx = canvas.getContext('2d');

const paddleWidth = 15, paddleHeight = 100, ballSize = 15;
const canvasWidth = canvas.width, canvasHeight = canvas.height;

// Game objects
let leftPaddle = {
    x: 10,
    y: canvasHeight / 2 - paddleHeight / 2,
    width: paddleWidth,
    height: paddleHeight,
    dy: 0,
    speed: 8
};

let rightPaddle = {
    x: canvasWidth - paddleWidth - 10,
    y: canvasHeight / 2 - paddleHeight / 2,
    width: paddleWidth,
    height: paddleHeight,
    dy: 0,
    speed: 6
};

let ball = {
    x: canvasWidth / 2 - ballSize / 2,
    y: canvasHeight / 2 - ballSize / 2,
    size: ballSize,
    dx: Math.random() > 0.5 ? 5 : -5,
    dy: (Math.random() - 0.5) * 8
};

let score = [0, 0];

// Draw functions
function drawRect(x, y, w, h, color='#fff') {
    ctx.fillStyle = color;
    ctx.fillRect(x, y, w, h);
}

function drawBall(x, y, size, color='#FFD700') {
    ctx.fillStyle = color;
    ctx.fillRect(x, y, size, size);
}

function drawNet() {
    for (let i = 0; i < canvasHeight; i += 30) {
        drawRect(canvasWidth/2 - 2, i, 4, 15, '#555');
    }
}

function draw() {
    ctx.clearRect(0, 0, canvasWidth, canvasHeight);
    drawNet();
    drawRect(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height);
    drawRect(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height);
    drawBall(ball.x, ball.y, ball.size);
}

function resetBall() {
    ball.x = canvasWidth / 2 - ballSize / 2;
    ball.y = canvasHeight / 2 - ballSize / 2;
    ball.dx = Math.random() > 0.5 ? 5 : -5;
    ball.dy = (Math.random() - 0.5) * 8;
}

// Update functions
function updateLeftPaddle() {
    leftPaddle.y += leftPaddle.dy;
    if (leftPaddle.y < 0) leftPaddle.y = 0;
    if (leftPaddle.y + leftPaddle.height > canvasHeight) leftPaddle.y = canvasHeight - leftPaddle.height;
}

function updateRightPaddle() {
    // Simple AI: follow the ball with some delay
    let paddleCenter = rightPaddle.y + rightPaddle.height / 2;
    if (ball.y + ball.size/2 < paddleCenter - 10) {
        rightPaddle.y -= rightPaddle.speed;
    } else if (ball.y + ball.size/2 > paddleCenter + 10) {
        rightPaddle.y += rightPaddle.speed;
    }
    // Clamp
    if (rightPaddle.y < 0) rightPaddle.y = 0;
    if (rightPaddle.y + rightPaddle.height > canvasHeight) rightPaddle.y = canvasHeight - rightPaddle.height;
}

function updateBall() {
    ball.x += ball.dx;
    ball.y += ball.dy;

    // Top/bottom wall collision
    if (ball.y < 0) {
        ball.y = 0;
        ball.dy *= -1;
    }
    if (ball.y + ball.size > canvasHeight) {
        ball.y = canvasHeight - ball.size;
        ball.dy *= -1;
    }

    // Left paddle collision
    if (ball.x < leftPaddle.x + leftPaddle.width &&
        ball.x > leftPaddle.x &&
        ball.y + ball.size > leftPaddle.y &&
        ball.y < leftPaddle.y + leftPaddle.height) {
        ball.x = leftPaddle.x + leftPaddle.width;
        ball.dx *= -1;
        // Add a bit of "spin"
        let collidePoint = (ball.y + ball.size/2) - (leftPaddle.y + leftPaddle.height/2);
        ball.dy = collidePoint * 0.2;
    }

    // Right paddle collision
    if (ball.x + ball.size > rightPaddle.x &&
        ball.x + ball.size < rightPaddle.x + rightPaddle.width &&
        ball.y + ball.size > rightPaddle.y &&
        ball.y < rightPaddle.y + rightPaddle.height) {
        ball.x = rightPaddle.x - ball.size;
        ball.dx *= -1;
        let collidePoint = (ball.y + ball.size/2) - (rightPaddle.y + rightPaddle.height/2);
        ball.dy = collidePoint * 0.2;
    }

    // Score
    if (ball.x < 0) {
        score[1]++;
        updateScoreboard();
        resetBall();
    }
    if (ball.x + ball.size > canvasWidth) {
        score[0]++;
        updateScoreboard();
        resetBall();
    }
}

function updateScoreboard() {
    document.getElementById('score-left').textContent = score[0];
    document.getElementById('score-right').textContent = score[1];
}

// Input
document.addEventListener('keydown', (e) => {
    if (e.key === "ArrowUp") {
        leftPaddle.dy = -leftPaddle.speed;
    } else if (e.key === "ArrowDown") {
        leftPaddle.dy = leftPaddle.speed;
    }
});
document.addEventListener('keyup', (e) => {
    if (e.key === "ArrowUp" || e.key === "ArrowDown") {
        leftPaddle.dy = 0;
    }
});

// Mouse controls
canvas.addEventListener('mousemove', (e) => {
    // Get mouse position relative to canvas
    const rect = canvas.getBoundingClientRect();
    let y = e.clientY - rect.top - leftPaddle.height/2;
    if (y < 0) y = 0;
    if (y + leftPaddle.height > canvasHeight) y = canvasHeight - leftPaddle.height;
    leftPaddle.y = y;
});

// Main game loop
function gameLoop() {
    updateLeftPaddle();
    updateRightPaddle();
    updateBall();
    draw();
    requestAnimationFrame(gameLoop);
}

// Start
updateScoreboard();
gameLoop();

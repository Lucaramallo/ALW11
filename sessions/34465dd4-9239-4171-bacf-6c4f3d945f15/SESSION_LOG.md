**YOU** (Requester)

build a simple tetrix game and push it to github

---

**AgentLink** (Observer)

Round 1 — Independent analysis

---

**AgentLink** (Observer)

[REPO_INIT] GitHub repository linked.
Repo: https://github.com/Lucaramallo/ALW11.git
Branch: agentlink/session-34465dd4
Strategy: branch
Files indexed: 1

---

**Nexus-7** (Contributor)

**Problem:** Need a functional Tetris implementation with GitHub deployment pipeline. **Solution:** Build a vanilla JavaScript/Canvas Tetris game with modular architecture (game logic, rendering, input handling), initialize a Git repository, and push to GitHub with proper .gitignore and README. **Implementation:** Create `index.html` with Canvas element, implement core game loop with piece rotation/collision detection in `game.js`, add styling in `style.css`, then `git init → add → commit → git remote add origin [repo-url] → git push -u origin main`.

---

**Vortex-UI** (Contributor)

I'd recommend starting with a lightweight HTML5/Canvas implementation using vanilla JavaScript—this keeps the scope manageable while delivering a fully playable Tetris experience with clear separation of game logic, rendering, and input handling. For the GitHub workflow, structure it as a simple repo with an `index.html`, `game.js`, and `style.css`, then initialize git locally, commit with clear messages (e.g., "feat: core game loop and piece mechanics"), and push to a new public repository for easy sharing and future collaboration. This approach prioritizes user experience through responsive controls and visual feedback while maintaining clean, maintainable code that others can quickly understand and contribute to.

---

**AgentLink** (Observer)

Round 2 — Final round

---

**Nexus-7** (Observer)

[REPO_COMMIT] Nexus-7 (Contributor) committed sessions/34465dd4-9239-4171-bacf-6c4f3d945f15/contributions/nexus-7.md
Message: [Nexus-7] (Contributor): Round 1 output
SHA: 6b5a9a5

---

**Vortex-UI** (Observer)

[REPO_COMMIT] Vortex-UI (Contributor) committed sessions/34465dd4-9239-4171-bacf-6c4f3d945f15/contributions/vortex-ui.md
Message: [Vortex-UI] (Contributor): Round 1 output
SHA: c919935

---

**Nexus-7** (Contributor)

# DELIVERABLE

## FILE 1: index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tetris Game</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="game-wrapper">
            <canvas id="gameCanvas" width="300" height="600"></canvas>
            <div class="sidebar">
                <div class="info-box">
                    <h2>Next</h2>
                    <canvas id="nextCanvas" width="120" height="120"></canvas>
                </div>
                <div class="info-box">
                    <h3>Score: <span id="score">0</span></h3>
                    <h3>Lines: <span id="lines">0</span></h3>
                    <h3>Level: <span id="level">1</span></h3>
                </div>
                <button id="startBtn" class="btn">Start Game</button>
                <button id="pauseBtn" class="btn" disabled>Pause</button>
                <button id="resetBtn" class="btn">Reset</button>
                <div class="controls">
                    <p><strong>Controls:</strong></p>
                    <p>← → Move</p>
                    <p>↓ Soft Drop</p>
                    <p>Space Rotate</p>
                </div>
            </div>
        </div>
    </div>
    <script src="game.js"></script>
</body>
</html>
```

## FILE 2: game.js
```javascript
const COLS = 10;
const ROWS = 20;
const BLOCK_SIZE = 30;

const PIECES = {
    I: { shape: [[1, 1, 1, 1]], color: '#00f0f0' },
    O: { shape: [[1, 1], [1, 1]], color: '#f0f000' },
    T: { shape: [[0, 1, 0], [1, 1, 1]], color: '#a000f0' },
    S: { shape: [[0, 1, 1], [1, 1, 0]], color: '#00f000' },
    Z: { shape: [[1, 1, 0], [0, 1, 1]], color: '#f00000' },
    J: { shape: [[1, 0, 0], [1, 1, 1]], color: '#0000f0' },
    L: { shape: [[0, 0, 1], [1, 1, 1]], color: '#f0a000' }
};

class Tetris {
    constructor() {
        this.canvas = document.getElementById('gameCanvas');
        this.ctx = this.canvas.getContext('2d');
        this.nextCanvas = document.getElementById('nextCanvas');
        this.nextCtx = this.nextCanvas.getContext('2d');
        
        this.grid = Array(ROWS).fill(null).map(() => Array(COLS).fill(0));
        this.currentPiece = null;
        this.nextPiece = null;
        this.score = 0;
        this.lines = 0;
        this.level = 1;
        this.gameActive = false;
        this.gamePaused = false;
        this.dropCounter = 0;
        this.dropInterval = 800;
        this.lastTime = 0;
        
        this.setupEventListeners();
        this.spawnPiece();
    }
    
    setupEventListeners() {
        document.getElementById('startBtn').addEventListener('click', () => this.start());
        document.getElementById('pauseBtn').addEventListener('click', () => this.togglePause());
        document.getElementById('resetBtn').addEventListener('click', () => this.reset());
        
        document.addEventListener('keydown', (e) => this.handleInput(e));
    }
    
    handleInput(e) {
        if (!this.gameActive || this.gamePaused) return;
        
        switch(e.key) {
            case 'ArrowLeft':
                this.moveLeft();
                e.preventDefault();
                break;
            case 'ArrowRight':
                this.moveRight();
                e.preventDefault();
                break;
            case 'ArrowDown':
                this.softDrop();
                e.preventDefault();
                break;
            case ' ':
                this.rotate();
                e.preventDefault();
                break;
        }
    }
    
    spawnPiece() {
        if (!this.nextPiece) {
            this.nextPiece = this.randomPiece();
        }
        this.currentPiece = this.nextPiece;
        this.nextPiece = this.randomPiece();
        
        this.currentPiece.x = Math.floor(COLS / 2) - 1;
        this.currentPiece.y = 0;
        
        if (this.collision(this.currentPiece)) {
            this.endGame();
        }
        this.drawNextPiece();
    }
    
    randomPiece() {
        const keys = Object.keys(PIECES);
        const key = keys[Math.floor(Math.random() * keys.length)];
        const piece = PIECES[key];
        return { shape: piece.shape, color: piece.color, x: 0, y: 0 };
    }
    
    moveLeft() {
        this.currentPiece.x--;
        if (this.collision(this.currentPiece)) {
            this.currentPiece.x++;
        }
    }
    
    moveRight() {
        this.currentPiece.x++;
        if (this.collision(this.currentPiece)) {
            this.currentPiece.x--;
        }
    }
    
    softDrop() {
        this.currentPiece.y++;
        if (this.collision(this.currentPiece)) {
            this.currentPiece.y--;
            this.lockPiece();
        }
    }
    
    rotate() {
        const original = this.currentPiece.shape;
        this.currentPiece.shape = this.rotateShape(this.currentPiece.shape);
        
        if (this.collision(this.currentPiece)) {
            this.currentPiece.shape = original;
        }
    }
    
    rotateShape(shape) {
        const N = shape.length;
        const M = shape[0].length;
        const rotated = Array(M).fill(null).map(() => Array(N).fill(0));
        
        for (let i = 0; i < N; i++) {
            for (let j = 0; j < M; j++) {
                rotated[j][N - 1 - i] = shape[i][j];
            }
        }
        return rotated;
    }
    
    collision(piece) {
        const shape = piece.shape;
        for (let row = 0; row < shape.length; row++) {
            for (let col = 0; col < shape[row].length; col++) {
                if (shape[row][col]) {
                    const x = piece.x + col;
                    const y = piece.y + row;
                    
                    if (x < 0 || x >= COLS || y >= ROWS) return true;
                    if (y >= 0 && this.grid[y][x]) return true;
                }
            }
        }
        return false;
    }
    
    lockPiece() {
        const shape = this.currentPiece.shape;
        for (let row = 0; row < shape.length; row++) {
            for (let col = 0; col < shape[row].length; col++) {
                if (shape[row][col]) {
                    const x = this.currentPiece.x + col;
                    const y = this.currentPiece.y + row;
                    if (y >= 0) {
                        this.grid[y][x] = this.currentPiece.color;
                    }
                }
            }
        }
        this.clearLines();
        this.spawnPiece();
    }
    
    clearLines() {
        let linesCleared = 0;
        for (let row = ROWS - 1; row >= 0; row--) {
            if (this.grid[row].every(cell => cell !== 0)) {
                this.grid.splice(row, 1);
                this.grid.unshift(Array(COLS).fill(0));
                linesCleared++;
                row++;
            }
        }
        if (linesCleared > 0) {
            this.lines += linesCleared;
            this.score += linesCleared * 100 * this.level;
            this.level = Math.floor(this.lines / 10) + 1;
            this.dropInterval = Math.max(100, 800 - (this.level - 1) * 50);
            this.updateStats();
        }
    }
    
    update(deltaTime) {
        this.dropCounter += deltaTime;
        
        if (this.dropCounter > this.dropInterval) {
            this.currentPiece.y++;
            if (this.collision(this.currentPiece)) {
                this.currentPiece.y--;
                this.lockPiece();
            }
            this.dropCounter = 0;
        }
    }
    
    draw() {
        this.ctx.fillStyle = '#000';
        this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
        
        this.ctx.strokeStyle = '#333';
        for (let i = 0; i <= COLS; i++) {
            this.ctx.beginPath();
            this.ctx.moveTo(i * BLOCK_SIZE, 0);
            this.ctx.lineTo(i * BLOCK_SIZE, this.canvas.height);
            this.ctx.stroke();
        }
        for (let i = 0; i <= ROWS; i++) {
            this.ctx.beginPath();
            this.ctx.moveTo(0, i * BLOCK_SIZE);
            this.ctx.lineTo(this.canvas.width, i * BLOCK_SIZE);
            this.ctx.stroke();
        }
        
        for (let row = 0; row < ROWS; row++) {
            for (let col = 0; col < COLS; col++) {
                if (this.grid[row][col]) {
                    this.ctx.fillStyle = this.grid[row][col];
                    this.ctx.fillRect(col * BLOCK_SIZE + 1, row * BLOCK_SIZE + 1, BLOCK_SIZE - 2, BLOCK_SIZE - 2);
                }
            }
        }
        
        this.drawPiece(this.currentPiece, this.ctx, 0, 0);
    }
    
    drawPiece(piece, context, offsetX, offsetY) {
        context.fillStyle = piece.color;
        const shape = piece.shape;
        for (let row = 0; row < shape.length; row++) {
            for (let col = 0; col < shape[row].length; col++) {
                if (shape[row][col]) {
                    context.fillRect(
                        (piece.x + col) * BLOCK_SIZE + offsetX + 1,
                        (piece.y + row) * BLOCK_SIZE + offsetY + 1,
                        BLOCK_SIZE - 2,
                        BLOCK_SIZE - 2
                    );
                }
            }
        }
    }
    
    drawNextPiece() {
        this.nextCtx.fillStyle = '#000';
        this.nextCtx.fillRect(0, 0, this.nextCanvas.width, this.nextCanvas.height);
        
        const tempPiece = { ...this.nextPiece, x: 1, y: 1 };
        this.drawPiece(tempPiece, this.nextCtx, 0, 0);
    }
    
    updateStats() {
        document.getElementById('score').textContent = this.score;
        document.getElementById('lines').textContent = this.lines;
        document.getElementById('level').textContent = this.level;
    }
    
    start() {
        this.gameActive = true;
        this.gamePaused = false;
        document.getElementById('startBtn').disabled = true;
        document.getElementById('pauseBtn').disabled = false;
        this.gameLoop();
    }
    
    togglePause() {
        this.gamePaused = !this.gamePaused;
        document.getElementById('pauseBtn').textContent = this.gamePaused ? 'Resume' : 'Pause';
        if (!this.gamePaused) this.gameLoop();
    }
    
    endGame() {
        this.gameActive = false;
        alert(`Game Over!\nScore: ${this.score}\nLines: ${this.lines}`);
        document.getElementById('startBtn').disabled = false;
        document.getElementById('pauseBtn').disabled = true;
    }
    
    reset() {
        this.grid = Array(ROWS).fill(null).map(() => Array(COLS).fill(0));
        this.score = 0;
        this.lines = 0;
        this.level = 1;
        this.gameActive = false;
        this.gamePaused = false;
        this.dropInterval = 800;
        this.updateStats();
        this.spawnPiece();
        this.draw();
        document.getElementById('startBtn').disabled = false;
        document.getElementById('pauseBtn').disabled = true;
        document.getElementById('pauseBtn').textContent = 'Pause';
    }
    
    gameLoop = (currentTime = 0) => {
        if (!this.lastTime) this.lastTime = currentTime;
        const deltaTime = currentTime - this.lastTime;
        this.lastTime = currentTime;
        
        if (!this.gamePaused) {
            this.update(deltaTime);
            this.draw();
        }
        
        if (this.gameActive) {
            requestAnimationFrame(this.gameLoop);
        }
    }
}

const game = new Tetris();
game.draw();
```

## FILE 3: style.css
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding: 20px;
}

.container {
    width: 100%;
    max-width: 600px;
}

.game-wrapper {
    display: flex;
    gap: 20px;
    background: #111;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
}

#gameCanvas {
    border: 3px solid #00f0f0;
    background: #000;
    image-rendering: pixelated;
    image-rendering: crisp-edges;
}

.sidebar {
    display: flex;
    flex-direction: column;
    gap: 15px;
    min-width: 150px;
}

.info-box {
    background: #222;
    padding: 15px;
    border-radius: 4px;
    border: 2px solid #00f0f0;
}

.info-box h2 {
    color: #00f0f0;
    font-size: 14px;
    margin-bottom: 10px;
}

.info-box h3 {
    color: #fff;
    font-size: 14px

---

**Nexus-7** (Observer)

[REPO_COMMIT] Nexus-7 (Contributor) committed sessions/34465dd4-9239-4171-bacf-6c4f3d945f15/contributions/nexus-7.md
Message: [Nexus-7] (Contributor): Round 2 output
SHA: 3643d90
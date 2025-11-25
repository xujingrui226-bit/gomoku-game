<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>五子棋 - 人机对战版</title>
    <style>
        body {
            margin: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: #f0f2f5;
            font-family: "Microsoft YaHei", sans-serif;
            user-select: none;
        }

        h1 {
            margin-bottom: 10px;
            color: #333;
            letter-spacing: 2px;
        }

        .game-info {
            margin-bottom: 15px;
            font-size: 18px;
            font-weight: bold;
            display: flex;
            gap: 20px;
            align-items: center;
        }

        .status-box {
            padding: 8px 20px;
            border-radius: 20px;
            background-color: #fff;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            transition: all 0.3s;
            border: 2px solid transparent;
        }

        /* 状态显示样式 */
        .status-player {
            border-color: #333;
            background-color: #333;
            color: white;
        }

        .status-ai {
            border-color: #888;
            color: #555;
        }

        canvas {
            background-color: #e4c690;
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
            border-radius: 4px;
            cursor: pointer; /* 默认指针 */
        }
        
        /* AI思考时禁用鼠标点击 */
        canvas.thinking {
            cursor: not-allowed;
        }

        .btn-group {
            margin-top: 20px;
        }

        button {
            padding: 10px 30px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            box-shadow: 0 4px #2E7D32;
            transition: all 0.1s;
        }

        button:active {
            box-shadow: 0 2px #2E7D32;
            transform: translateY(2px);
        }
    </style>
</head>
<body>

    <h1>五子棋 - 人机对战</h1>

    <div class="game-info">
        <div id="statusMsg" class="status-box status-player">👤 玩家回合 (黑棋)</div>
    </div>

    <canvas id="chessBoard" width="450" height="450"></canvas>

    <div class="btn-group">
        <button onclick="initGame()">重新开始</button>
    </div>

<script>
    const canvas = document.getElementById('chessBoard');
    const ctx = canvas.getContext('2d');
    const statusMsg = document.getElementById('statusMsg');

    // 配置参数
    const GRID_SIZE = 15;
    const CELL_SIZE = 30;
    const PADDING = 15;
    
    // 游戏状态
    let board = [];         // 0:空, 1:玩家(黑), 2:电脑(白)
    let isPlayerTurn = true;
    let isGameOver = false;

    // 初始化
    function initGame() {
        isPlayerTurn = true;
        isGameOver = false;
        board = [];
        
        // 初始化空棋盘
        for(let i=0; i<GRID_SIZE; i++){
            board[i] = [];
            for(let j=0; j<GRID_SIZE; j++){
                board[i][j] = 0;
            }
        }
        
        updateStatusUI();
        drawBoard();
        canvas.classList.remove('thinking');
    }

    // 更新状态栏文字
    function updateStatusUI() {
        if (isGameOver) return;
        if (isPlayerTurn) {
            statusMsg.innerText = "👤 玩家回合 (黑棋)";
            statusMsg.className = "status-box status-player";
        } else {
            statusMsg.innerText = "🤖 AI正在思考...";
            statusMsg.className = "status-box status-ai";
        }
    }

    // 绘制棋盘
    function drawBoard() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        ctx.lineWidth = 1;
        ctx.strokeStyle = "#5a3d1b"; 

        // 画线
        for (let i = 0; i < GRID_SIZE; i++) {
            ctx.beginPath();
            ctx.moveTo(PADDING, PADDING + i * CELL_SIZE);
            ctx.lineTo(canvas.width - PADDING, PADDING + i * CELL_SIZE);
            ctx.stroke();
            
            ctx.beginPath();
            ctx.moveTo(PADDING + i * CELL_SIZE, PADDING);
            ctx.lineTo(PADDING + i * CELL_SIZE, canvas.height - PADDING);
            ctx.stroke();
        }

        // 画星位
        const stars = [[3,3], [11,3], [7,7], [3,11], [11,11]];
        ctx.fillStyle = "#5a3d1b";
        stars.forEach(point => {
            ctx.beginPath();
            ctx.arc(PADDING + point[0]*CELL_SIZE, PADDING + point[1]*CELL_SIZE, 3, 0, 2*Math.PI);
            ctx.fill();
        });

        // 绘制已有棋子
        for(let i=0; i<GRID_SIZE; i++){
            for(let j=0; j<GRID_SIZE; j++){
                if(board[i][j] !== 0){
                    drawPiece(i, j, board[i][j] === 1);
                }
            }
        }
    }

    // 绘制棋子
    function drawPiece(x, y, isBlack) {
        ctx.beginPath();
        const centerX = PADDING + x * CELL_SIZE;
        const centerY = PADDING + y * CELL_SIZE;
        ctx.arc(centerX, centerY, 13, 0, 2 * Math.PI);
        
        const gradient = ctx.createRadialGradient(centerX - 3, centerY - 3, 1, centerX, centerY, 13);
        if (isBlack) {
            gradient.addColorStop(0, "#666");
            gradient.addColorStop(1, "#000");
        } else {
            gradient.addColorStop(0, "#fff");
            gradient.addColorStop(1, "#d1d1d1");
        }
        ctx.fillStyle = gradient;
        ctx.fill();
        ctx.shadowColor = "rgba(0, 0, 0, 0.4)";
        ctx.shadowBlur = 4;
        ctx.shadowOffsetX = 2;
        ctx.shadowOffsetY = 2;
        setTimeout(() => { ctx.shadowColor = "transparent"; }, 0);
    }

    // 玩家点击事件
    canvas.onclick = function(e) {
        if (isGameOver || !isPlayerTurn) return;

        const rect = canvas.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;

        const i = Math.round((x - PADDING) / CELL_SIZE);
        const j = Math.round((y - PADDING) / CELL_SIZE);

        if (i < 0 || i >= GRID_SIZE || j < 0 || j >= GRID_SIZE) return;
        if (board[i][j] !== 0) return;

        // 玩家落子
        board[i][j] = 1;
        drawPiece(i, j, true);

        if (checkWin(i, j, 1)) {
            endGame("🎉 恭喜！你赢了！");
            return;
        }

        // 切换到AI
        isPlayerTurn = false;
        canvas.classList.add('thinking');
        updateStatusUI();

        // 延迟一下，让玩家看到落子，再执行AI计算
        setTimeout(computerMove, 300);
    };

    // 游戏结束
    function endGame(msg) {
        isGameOver = true;
        statusMsg.innerText = msg;
        setTimeout(() => alert(msg), 50);
    }

    // --- AI 核心逻辑 ---
    function computerMove() {
        if (isGameOver) return;

        // 1. 计算所有空位的得分
        let maxScore = -1;
        let bestPoints = [];

        for (let i = 0; i < GRID_SIZE; i++) {
            for (let j = 0; j < GRID_SIZE; j++) {
                if (board[i][j] === 0) {
                    // 评估该点的价值：(我方进攻分 + 敌方威胁分)
                    // 2代表电脑，1代表玩家
                    let score = evaluatePoint(i, j, 2) + evaluatePoint(i, j, 1);
                    
                    if (score > maxScore) {
                        maxScore = score;
                        bestPoints = [{x: i, y: j}];
                    } else if (score === maxScore) {
                        bestPoints.push({x: i, y: j});
                    }
                }
            }
        }

        // 2. 选取最高分的位置落子 (如果有多个，随机选一个)
        if (bestPoints.length > 0) {
            const move = bestPoints[Math.floor(Math.random() * bestPoints.length)];
            board[move.x][move.y] = 2; // 电脑落白子
            drawPiece(move.x, move.y, false);

            if (checkWin(move.x, move.y, 2)) {
                endGame("💻 电脑赢了，再接再厉！");
                return;
            }
        }

        // 3. 切换回玩家
        isPlayerTurn = true;
        canvas.classList.remove('thinking');
        updateStatusUI();
    }

    // 评估某一个空位的分数
    // role: 2=电脑(计算进攻分), 1=玩家(计算防守分)
    function evaluatePoint(x, y, role) {
        let totalScore = 0;
        // 四个方向
        const directions = [[[1,0], [-1,0]], [[0,1], [0,-1]], [[1,1], [-1,-1]], [[1,-1], [-1,1]]];

        for (let dir of directions) {
            let count = 1;  // 连子数
            let empty = 0;  // 两端是否有空位

            // 向两个方向延伸
            for (let side of dir) {
                let dx = side[0];
                let dy = side[1];
                let nx = x + dx;
                let ny = y + dy;
                
                // 连续同色棋子
                while (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && board[nx][ny] === role) {
                    count++;
                    nx += dx;
                    ny += dy;
                }
                
                // 检查尽头是否为空位
                if (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && board[nx][ny] === 0) {
                    empty++;
                }
            }

            // --- 评分规则 ---
            // 这里的权重非常重要：
            // 连5 > 活4 > 冲4/活3 > 活2
            
            if (count >= 5) return 100000; // 连五 (必胜/必防)
            
            if (count === 4) {
                if (empty === 2) totalScore += 10000; // 活四 (两头空)
                else if (empty === 1) totalScore += 2500; // 冲四 (被堵一头)
            } 
            else if (count === 3) {
                if (empty === 2) totalScore += 2500; // 活三 (威胁大)
                else if (empty === 1) totalScore += 500;  // 眠三
            }
            else if (count === 2) {
                if (empty === 2) totalScore += 100; // 活二
            }
        }
        
        // 如果是电脑自己(进攻)，稍微增加一点权重，鼓励进攻
        if(role === 2) totalScore *= 1.1; 
        
        return totalScore;
    }

    // 胜负判断
    function checkWin(x, y, color) {
        const directions = [[[1,0], [-1,0]], [[0,1], [0,-1]], [[1,1], [-1,-1]], [[1,-1], [-1,1]]];
        for (let dir of directions) {
            let count = 1;
            for (let side of dir) {
                let nx = x + side[0];
                let ny = y + side[1];
                while (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && board[nx][ny] === color) {
                    count++;
                    nx += side[0];
                    ny += side[1];
                }
            }
            if (count >= 5) return true;
        }
        return false;
    }

    // 启动
    initGame();
</script>
</body>
</html>

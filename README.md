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
    // AI配置
    const AI_DEPTH = 3;     // 搜索深度，越大AI越强但思考越慢
    const WIN_SCORE = 1000000;  // 获胜分数

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

    // --- 增强版AI 核心逻辑 ---
    function computerMove() {
        if (isGameOver) return;

        // 使用极小极大算法带α-β剪枝寻找最佳落子
        let bestScore = -Infinity;
        let bestMove = null;
        let alpha = -Infinity;
        const beta = Infinity;

        // 先生成所有可能的落子点并排序（优先考虑已有棋子周围）
        const candidates = getCandidateMoves();
        
        for (const move of candidates) {
            const {x, y} = move;
            board[x][y] = 2; // 尝试落子
            
            // 递归评估
            const score = minimax(AI_DEPTH - 1, false, alpha, beta);
            
            board[x][y] = 0; // 回溯
            
            if (score > bestScore) {
                bestScore = score;
                bestMove = {x, y};
                alpha = Math.max(alpha, score);
            }
        }

        // 执行最佳落子
        if (bestMove) {
            board[bestMove.x][bestMove.y] = 2;
            drawPiece(bestMove.x, bestMove.y, false);

            if (checkWin(bestMove.x, bestMove.y, 2)) {
                endGame("💻 电脑赢了，再接再厉！");
                return;
            }
        }

        // 切换回玩家
        isPlayerTurn = true;
        canvas.classList.remove('thinking');
        updateStatusUI();
    }

    // 极小极大算法带α-β剪枝
    function minimax(depth, isMaximizing, alpha, beta) {
        // 检查当前局面是否有胜负
        if (hasWin(2)) return WIN_SCORE + depth; // AI赢了，深度越深加分越少
        if (hasWin(1)) return -WIN_SCORE - depth; // 玩家赢了
        if (depth === 0) return evaluateBoard(); // 到达最大深度，评估当前局面
        
        // 获取候选落子点
        const candidates = getCandidateMoves();
        if (candidates.length === 0) return 0; // 平局
        
        if (isMaximizing) {
            // AI回合，最大化分数
            let maxScore = -Infinity;
            for (const move of candidates) {
                const {x, y} = move;
                board[x][y] = 2;
                const score = minimax(depth - 1, false, alpha, beta);
                board[x][y] = 0;
                
                maxScore = Math.max(maxScore, score);
                alpha = Math.max(alpha, score);
                
                if (beta <= alpha) break; // α-β剪枝
            }
            return maxScore;
        } else {
            // 玩家回合，最小化分数
            let minScore = Infinity;
            for (const move of candidates) {
                const {x, y} = move;
                board[x][y] = 1;
                const score = minimax(depth - 1, true, alpha, beta);
                board[x][y] = 0;
                
                minScore = Math.min(minScore, score);
                beta = Math.min(beta, score);
                
                if (beta <= alpha) break; // α-β剪枝
            }
            return minScore;
        }
    }

    // 获取候选落子点（优先考虑已有棋子周围）
    function getCandidateMoves() {
        const candidates = [];
        const weights = [];
        
        // 遍历棋盘
        for (let i = 0; i < GRID_SIZE; i++) {
            for (let j = 0; j < GRID_SIZE; j++) {
                if (board[i][j] === 0) {
                    // 检查周围是否有棋子，有棋子的位置优先级高
                    let weight = 0;
                    for (let dx = -2; dx <= 2; dx++) {
                        for (let dy = -2; dy <= 2; dy++) {
                            if (dx === 0 && dy === 0) continue;
                            const nx = i + dx;
                            const ny = j + dy;
                            if (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && board[nx][ny] !== 0) {
                                weight++;
                            }
                        }
                    }
                    
                    if (weight > 0 || candidates.length < 10) { // 至少保留10个点
                        candidates.push({x: i, y: j});
                        weights.push(weight);
                    }
                }
            }
        }
        
        // 根据权重排序，权重高的先搜索（提高剪枝效率）
        candidates.sort((a, b) => {
            const idxA = candidates.indexOf(a);
            const idxB = candidates.indexOf(b);
            return weights[idxB] - weights[idxA];
        });
        
        return candidates;
    }

    // 检查是否有一方获胜
    function hasWin(role) {
        for (let i = 0; i < GRID_SIZE; i++) {
            for (let j = 0; j < GRID_SIZE; j++) {
                if (board[i][j] === role && checkWin(i, j, role)) {
                    return true;
                }
            }
        }
        return false;
    }

    // 评估当前棋盘分数
    function evaluateBoard() {
        let score = 0;
        
        // 评估每个位置对双方的价值
        for (let i = 0; i < GRID_SIZE; i++) {
            for (let j = 0; j < GRID_SIZE; j++) {
                if (board[i][j] === 0) {
                    // AI的进攻分减去玩家的进攻分（防守）
                    score += evaluatePoint(i, j, 2) - evaluatePoint(i, j, 1) * 0.95;
                }
            }
        }
        
        return score;
    }

    // 评估某一个空位的分数
    // role: 2=电脑(计算进攻分), 1=玩家(计算防守分)
    function evaluatePoint(x, y, role) {
        let totalScore = 0;
        // 四个方向
        const directions = [[[1,0], [-1,0]], [[0,1], [0,-1]], [[1,1], [-1,-1]], [[1,-1], [-1,1]]];

        for (let dir of directions) {
            // 计算连续的棋子和阻塞情况
            let [count, blocks, empty] = analyzeLine(x, y, role, dir);
            
            // 更细致的评分规则
            if (count >= 5) return WIN_SCORE / 2; // 连五
            
            // 活四（两端都不堵）
            if (count === 4 && blocks === 0) totalScore += 100000;
            // 冲四（一端堵，另一端不堵）
            else if (count === 4 && blocks === 1) totalScore += 10000;
            
            // 活三（两端不堵）
            else if (count === 3 && blocks === 0) totalScore += 5000;
            // 冲三（一端堵）
            else if (count === 3 && blocks === 1) totalScore += 1000;
            
            // 活二（两端不堵）
            else if (count === 2 && blocks === 0) totalScore += 500;
            // 冲二（一端堵）
            else if (count === 2 && blocks === 1) totalScore += 100;
            
            // 单棋
            else if (count === 1 && blocks === 0) totalScore += 10;
        }
        
        return totalScore;
    }

    // 分析一条线上的棋子情况
    function analyzeLine(x, y, role, dir) {
        let count = 1; // 包括当前点
        let blocks = 0; // 阻塞数
        let empty = 0; // 空位数
        
        // 检查两个方向
        for (let side of dir) {
            let dx = side[0];
            let dy = side[1];
            let nx = x + dx;
            let ny = y + dy;
            let currentCount = 0;
            
            // 计算连续的同色棋子
            while (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && board[nx][ny] === role) {
                currentCount++;
                nx += dx;
                ny += dy;
            }
            
            count += currentCount;
            
            // 检查是否被阻塞
            if (nx < 0 || nx >= GRID_SIZE || ny < 0 || ny >= GRID_SIZE) {
                // 边界视为阻塞
                blocks++;
            } else if (board[nx][ny] !== 0) {
                // 被对方棋子阻塞
                blocks++;
            } else {
                // 空位
                empty++;
            }
        }
        
        return [count, blocks, empty];
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

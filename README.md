# Chess-game
<!DOCTYPE html>
<html lang="ku">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>شەترەنجی هاوڕێکان</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #1a1a2e, #16213e, #0f3460);
            font-family: 'Arial', sans-serif;
            color: white;
        }

        .game-container {
            display: flex;
            gap: 30px;
            max-width: 1200px;
            width: 100%;
        }

        .chess-section {
            flex: 2;
        }

        .sidebar {
            flex: 1;
            background: rgba(0, 0, 0, 0.3);
            padding: 20px;
            border-radius: 10px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .chessboard {
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1/1;
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            border-radius: 8px;
            overflow: hidden;
            margin: 0 auto;
        }

        .square {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 30px;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
        }

        .light {
            background-color: #f0d9b5;
        }

        .dark {
            background-color: #b58863;
        }

        .selected {
            background-color: rgba(103, 255, 103, 0.6) !important;
        }

        .possible-move {
            background-color: rgba(255, 255, 103, 0.6) !important;
        }

        .possible-capture {
            background-color: rgba(255, 103, 103, 0.6) !important;
        }

        .controls {
            display: flex;
            gap: 15px;
            justify-content: center;
            margin-top: 20px;
        }

        button {
            padding: 10px 20px;
            background: linear-gradient(45deg, #ff6b6b, #ff8e8e);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
            transition: all 0.3s ease;
        }

        button:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.3);
        }

        h1 {
            text-align: center;
            text-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
            margin-bottom: 20px;
        }

        .player-info {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }

        .timer {
            font-size: 24px;
            font-weight: bold;
            text-align: center;
            margin: 10px 0;
        }

        .player-white {
            color: #f0d9b5;
        }

        .player-black {
            color: #b58863;
        }

        .invite-section {
            background: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 8px;
        }

        input {
            padding: 8px;
            border-radius: 4px;
            border: none;
            margin-right: 10px;
            width: 70%;
        }

        .chat-box {
            height: 150px;
            overflow-y: auto;
            background: rgba(0, 0, 0, 0.2);
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 10px;
        }

        .chat-message {
            margin: 5px 0;
            padding: 5px;
            border-radius: 3px;
        }

        .friend-message {
            background: rgba(0, 100, 255, 0.2);
            text-align: left;
        }

        .my-message {
            background: rgba(0, 255, 100, 0.2);
            text-align: right;
        }

        .game-status {
            text-align: center;
            font-weight: bold;
            margin: 10px 0;
            min-height: 24px;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div class="chess-section">
            <h1>شەترەنجی هاوڕێکان</h1>
            <div class="game-status" id="gameStatus">پاشکەوتکردنی یارییەکە...</div>
            <div class="player-info">
                <div class="player-white" id="whitePlayer">سپی: تۆ</div>
                <div class="timer" id="whiteTimer">10:00</div>
            </div>
            <div class="chessboard" id="chessboard"></div>
            <div class="player-info">
                <div class="player-black" id="blackPlayer">ڕەش: چاوەڕوانی هاوڕێ</div>
                <div class="timer" id="blackTimer">10:00</div>
            </div>
            <div class="controls">
                <button id="reset">دووبارەکردنەوە</button>
                <button id="flip">سەرەوخوارکردن</button>
                <button id="offerDraw">پێشنیازی بەرامبەر</button>
            </div>
        </div>
        
        <div class="sidebar">
            <div class="invite-section">
                <h3>داوای هاوڕێ</h3>
                <div>
                    <input type="text" id="friendCode" placeholder="کۆدی هاوڕێ" readonly>
                    <button id="copyCode">کۆپی کۆد</button>
                </div>
                <div style="margin-top: 10px;">
                    <input type="text" id="joinCode" placeholder="کۆدی یاری">
                    <button id="joinGame">بچۆ ناو یاریەکە</button>
                </div>
            </div>
            
            <div class="chat-box" id="chatBox"></div>
            <div>
                <input type="text" id="chatInput" placeholder="نامە بنێرە...">
                <button id="sendMessage">ناردن</button>
            </div>
            
            <div class="game-status" id="connectionStatus">پەیوەندی نەکراوە</div>
        </div>
    </div>

    <script>
        // Game Elements
        const chessboard = document.getElementById('chessboard');
        const resetBtn = document.getElementById('reset');
        const flipBtn = document.getElementById('flip');
        const offerDrawBtn = document.getElementById('offerDraw');
        const whiteTimer = document.getElementById('whiteTimer');
        const blackTimer = document.getElementById('blackTimer');
        const gameStatus = document.getElementById('gameStatus');
        const whitePlayer = document.getElementById('whitePlayer');
        const blackPlayer = document.getElementById('blackPlayer');
        const connectionStatus = document.getElementById('connectionStatus');

        // Multiplayer Elements
        const friendCode = document.getElementById('friendCode');
        const copyCodeBtn = document.getElementById('copyCode');
        const joinCode = document.getElementById('joinCode');
        const joinGameBtn = document.getElementById('joinGame');
        const chatBox = document.getElementById('chatBox');
        const chatInput = document.getElementById('chatInput');
        const sendMessageBtn = document.getElementById('sendMessage');

        // Game State
        const initialBoard = [
            ['♜', '♞', '♝', '♛', '♚', '♝', '♞', '♜'],
            ['♟', '♟', '♟', '♟', '♟', '♟', '♟', '♟'],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['♙', '♙', '♙', '♙', '♙', '♙', '♙', '♙'],
            ['♖', '♘', '♗', '♕', '♔', '♗', '♘', '♖']
        ];

        let currentBoard = JSON.parse(JSON.stringify(initialBoard));
        let isFlipped = false;
        let selectedSquare = null;
        let currentTurn = 'white';
        let possibleMoves = [];
        let gameId = null;
        let playerColor = 'white';
        let playerName = "Player_" + Math.floor(Math.random() * 1000);
        let opponentName = "هاوڕێ";
        let whiteTime = 600; // 10 minutes in seconds
        let blackTime = 600;
        let timerInterval = null;
        let connection = null;

        // Initialize
        function init() {
            generateGameCode();
            setupEventListeners();
            createBoard();
            startTimer();
        }

        // Generate a simple game code
        function generateGameCode() {
            const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
            let code = '';
            for (let i = 0; i < 6; i++) {
                code += chars.charAt(Math.floor(Math.random() * chars.length));
            }
            friendCode.value = code;
            gameId = code;
        }

        // Setup event listeners
        function setupEventListeners() {
            // Game controls
            resetBtn.addEventListener('click', resetGame);
            flipBtn.addEventListener('click', flipBoard);
            offerDrawBtn.addEventListener('click', offerDraw);
            
            // Multiplayer controls
            copyCodeBtn.addEventListener('click', copyGameCode);
            joinGameBtn.addEventListener('click', joinGame);
            sendMessageBtn.addEventListener('click', sendChatMessage);
            chatInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') sendChatMessage();
            });
            
            // Simulate connection (in a real app, this would be WebSocket)
            setTimeout(() => {
                connectionStatus.textContent = "پەیوەندی کراوە";
                connectionStatus.style.color = "#4CAF50";
            }, 2000);
        }

        // Chess board functions
        function createBoard() {
            chessboard.innerHTML = '';
            for (let row = 0; row < 8; row++) {
                for (let col = 0; col < 8; col++) {
                    const square = document.createElement('div');
                    square.className = `square ${(row + col) % 2 === 0 ? 'light' : 'dark'}`;
                    
                    const displayRow = isFlipped ? 7 - row : row;
                    const displayCol = isFlipped ? 7 - col : col;
                    
                    square.textContent = currentBoard[displayRow][displayCol];
                    square.dataset.row = displayRow;
                    square.dataset.col = displayCol;

                    // Highlight possible moves
                    const isPossibleMove = possibleMoves.some(move => 
                        move.row == displayRow && move.col == displayCol
                    );
                    
                    if (isPossibleMove) {
                        const targetPiece = currentBoard[displayRow][displayCol];
                        if (targetPiece) {
                            square.classList.add('possible-capture');
                        } else {
                            square.classList.add('possible-move');
                        }
                    }

                    // Highlight selected square
                    if (selectedSquare && selectedSquare.row == displayRow && selectedSquare.col == displayCol) {
                        square.classList.add('selected');
                    }

                    // Only allow clicks if it's your turn
                    if (currentTurn === playerColor) {
                        square.addEventListener('click', () => handleSquareClick(displayRow, displayCol));
                    } else {
                        square.style.cursor = 'not-allowed';
                    }

                    chessboard.appendChild(square);
                }
            }

            // Update UI
            updatePlayerInfo();
            updateGameStatus();
        }

        function handleSquareClick(row, col) {
            const piece = currentBoard[row][col];
            
            // If no piece is selected and clicked on a piece of current turn's color
            if (!selectedSquare && piece && isCurrentTurnPiece(piece)) {
                selectedSquare = { row, col };
                possibleMoves = getPossibleMoves(row, col, piece);
                createBoard();
                return;
            }

            // If a piece is already selected
            if (selectedSquare) {
                // Check if clicked on a possible move
                const isMoveValid = possibleMoves.some(move => 
                    move.row === row && move.col === col
                );

                if (isMoveValid) {
                    // Move the piece
                    const [fromRow, fromCol] = [selectedSquare.row, selectedSquare.col];
                    const pieceMoved = currentBoard[fromRow][fromCol];
                    currentBoard[row][col] = pieceMoved;
                    currentBoard[fromRow][fromCol] = '';
                    
                    // Handle pawn promotion
                    if ((pieceMoved === '♙' && row === 0) || (pieceMoved === '♟' && row === 7)) {
                        currentBoard[row][col] = pieceMoved === '♙' ? '♕' : '♛'; // Auto-queen promotion
                    }
                    
                    // Switch turn
                    currentTurn = currentTurn === 'white' ? 'black' : 'white';
                    
                    // In a real game, you would send the move to the opponent here
                    sendMoveToOpponent(fromRow, fromCol, row, col);
                }

                // Reset selection
                selectedSquare = null;
                possibleMoves = [];
                createBoard();
            }
        }

        function getPossibleMoves(row, col, piece) {
            const moves = [];
            const isWhite = '♙♖♘♗♕♔'.includes(piece);

            // Pawn moves (fixed)
            if (piece === '♙' || piece === '♟') {
                const direction = isWhite ? -1 : 1;
                
                // Forward move
                if (isEmpty(row + direction, col)) {
                    moves.push({ row: row + direction, col });
                    
                    // Double move from starting position
                    if ((row === 6 && isWhite) || (row === 1 && !isWhite)) {
                        if (isEmpty(row + 2*direction, col) && isEmpty(row + direction, col)) {
                            moves.push({ row: row + 2*direction, col });
                        }
                    }
                }
                
                // Captures
                for (const captureCol of [col - 1, col + 1]) {
                    if (isOpponentPiece(row + direction, captureCol, isWhite)) {
                        moves.push({ row: row + direction, col: captureCol });
                    }
                }
            }

            // Rook moves
            if (piece === '♖' || piece === '♜') {
                const directions = [[1, 0], [-1, 0], [0, 1], [0, -1]];
                for (const [dr, dc] of directions) {
                    for (let i = 1; i < 8; i++) {
                        const newRow = row + i*dr;
                        const newCol = col + i*dc;
                        if (!isInBoard(newRow, newCol)) break;
                        if (isEmpty(newRow, newCol)) {
                            moves.push({ row: newRow, col: newCol });
                        } else {
                            if (isOpponentPiece(newRow, newCol, isWhite)) {
                                moves.push({ row: newRow, col: newCol });
                            }
                            break;
                        }
                    }
                }
            }

            // Knight moves
            if (piece === '♘' || piece === '♞') {
                const knightMoves = [
                    [2, 1], [2, -1], [-2, 1], [-2, -1],
                    [1, 2], [1, -2], [-1, 2], [-1, -2]
                ];
                for (const [dr, dc] of knightMoves) {
                    const newRow = row + dr;
                    const newCol = col + dc;
                    if (isInBoard(newRow, newCol) && 
                        (isEmpty(newRow, newCol) || isOpponentPiece(newRow, newCol, isWhite))) {
                        moves.push({ row: newRow, col: newCol });
                    }
                }
            }

            // Bishop moves
            if (piece === '♗' || piece === '♝') {
                const directions = [[1, 1], [1, -1], [-1, 1], [-1, -1]];
                for (const [dr, dc] of directions) {
                    for (let i = 1; i < 8; i++) {
                        const newRow = row + i*dr;
                        const newCol = col + i*dc;
                        if (!isInBoard(newRow, newCol)) break;
                        if (isEmpty(newRow, newCol)) {
                            moves.push({ row: newRow, col: newCol });
                        } else {
                            if (isOpponentPiece(newRow, newCol, isWhite)) {
                                moves.push({ row: newRow, col: newCol });
                            }
                            break;
                        }
                    }
                }
            }

            // Queen moves (rook + bishop)
            if (piece === '♕' || piece === '♛') {
                const directions = [
                    [1, 0], [-1, 0], [0, 1], [0, -1], // Rook
                    [1, 1], [1, -1], [-1, 1], [-1, -1] // Bishop
                ];
                for (const [dr, dc] of directions) {
                    for (let i = 1; i < 8; i++) {
                        const newRow = row + i*dr;
                        const newCol = col + i*dc;
                        if (!isInBoard(newRow, newCol)) break;
                        if (isEmpty(newRow, newCol)) {
                            moves.push({ row: newRow, col: newCol });
                        } else {
                            if (isOpponentPiece(newRow, newCol, isWhite)) {
                                moves.push({ row: newRow, col: newCol });
                            }
                            break;
                        }
                    }
                }
            }

            // King moves (simplified, no castling)
            if (piece === '♔' || piece === '♚') {
                for (let dr = -1; dr <= 1; dr++) {
                    for (let dc = -1; dc <= 1; dc++) {
                        if (dr === 0 && dc === 0) continue;
                        const newRow = row + dr;
                        const newCol = col + dc;
                        if (isInBoard(newRow, newCol) && 
                            (isEmpty(newRow, newCol) || isOpponentPiece(newRow, newCol, isWhite))) {
                            moves.push({ row: newRow, col: newCol });
                        }
                    }
                }
            }

            return moves;
        }

        // Helper functions
        function isEmpty(row, col) {
            return isInBoard(row, col) && currentBoard[row][col] === '';
        }

        function isOpponentPiece(row, col, isWhite) {
            if (!isInBoard(row, col)) return false;
            const piece = currentBoard[row][col];
            if (!piece) return false;
            return ('♙♖♘♗♕♔'.includes(piece) !== isWhite);
        }

        function isInBoard(row, col) {
            return row >= 0 && row < 8 && col >= 0 && col < 8;
        }

        function isCurrentTurnPiece(piece) {
            const isWhitePiece = '♙♖♘♗♕♔'.includes(piece);
            return (currentTurn === 'white' && isWhitePiece) || 
                   (currentTurn === 'black' && !isWhitePiece);
        }

        // Game control functions
        function resetGame() {
            currentBoard = JSON.parse(JSON.stringify(initialBoard));
            selectedSquare = null;
            possibleMoves = [];
            currentTurn = 'white';
            whiteTime = 600;
            blackTime = 600;
            updateTimers();
            createBoard();
            addChatMessage("سیستەم", "یارییەکە دووبارە کرایەوە!");
        }

        function flipBoard() {
            isFlipped = !isFlipped;
            createBoard();
        }

        function offerDraw() {
            addChatMessage("تۆ", "پێشنیازی بەرامبەرم کرد!");
            // In a real game, this would send to opponent
        }

        // Timer functions
        function startTimer() {
            if (timerInterval) clearInterval(timerInterval);
            timerInterval = setInterval(updateTimers, 1000);
        }

        function updateTimers() {
            if (currentTurn === 'white') {
                whiteTime--;
            } else {
                blackTime--;
            }

            // Check for time out
            if (whiteTime <= 0 || blackTime <= 0) {
                clearInterval(timerInterval);
                gameStatus.textContent = whiteTime <= 0 ? "ڕەش بردییەوە (کات)! " : "سپی بردییەوە (کات)! ";
                return;
            }

            // Update timer displays
            const formatTime = (seconds) => {
                const mins = Math.floor(seconds / 60);
                const secs = seconds % 60;
                return `${mins}:${secs < 10 ? '0' : ''}${secs}`;
            };

            whiteTimer.textContent = formatTime(whiteTime);
            blackTimer.textContent = formatTime(blackTime);
        }

        // Multiplayer functions
        function copyGameCode() {
            friendCode.select();
            document.execCommand('copy');
            addChatMessage("سیستەم", "کۆدی یاری کۆپی کراوە!");
        }

        function joinGame() {
            const code = joinCode.value.trim();
            if (code.length !== 6) {
                addChatMessage("سیستەم", "کۆدی یاری نادروستە!");
                return;
            }
            
            // In a real game, this would connect to the other player
            gameId = code;
            playerColor = 'black';
            opponentName = "Host";
            whitePlayer.textContent = `سپی: ${opponentName}`;
            blackPlayer.textContent = `ڕەش: تۆ`;
            isFlipped = true;
            
            addChatMessage("سیستەم", `پەیوەندی بە یاری ${code} کرد!`);
            createBoard();
        }

        function sendMoveToOpponent(fromRow, fromCol, toRow, toCol) {
            // In a real game, this would send the move via WebSocket
            addChatMessage("سیستەم", `جوڵە نێردرا بۆ هاوڕێ: ${String.fromCharCode(97+fromCol)}${8-fromRow} بۆ ${String.fromCharCode(97+toCol)}${8-toRow}`);
        }

        function sendChatMessage() {
            const message = chatInput.value.trim();
            if (message) {
                addChatMessage("تۆ", message);
                // In a real game, this would send to opponent
                chatInput.value = '';
            }
        }

        function addChatMessage(sender, message) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `chat-message ${sender === "تۆ" ? 'my-message' : 'friend-message'}`;
            messageDiv.innerHTML = `<strong>${sender}:</strong> ${message}`;
            chatBox.appendChild(messageDiv);
            chatBox.scrollTop = chatBox.scrollHeight;
        }

        // UI update functions
        function updatePlayerInfo() {
            if (playerColor === 'white') {
                whitePlayer.textContent = `سپی: تۆ`;
                blackPlayer.textContent = `ڕەش: ${opponentName}`;
            } else {
                whitePlayer.textContent = `سپی: ${opponentName}`;
                blackPlayer.textContent = `ڕەش: تۆ`;
            }
        }

        function updateGameStatus() {
            if (currentTurn === playerColor) {
                gameStatus.textContent = "نوێژەتەوە!";
                gameStatus.style.color = "#4CAF50";
            } else {
                gameStatus.textContent = "چاوەڕوانی هاوڕێ...";
                gameStatus.style.color = "#FF9800";
            }
        }

        // Initialize the game
        init();
    </script>
</body>
</html>

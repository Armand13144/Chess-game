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
        .chessboard {
            width: 400px;
            height: 400px;
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            border-radius: 8px;
            overflow: hidden;
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
        .light { background-color: #f0d9b5; }
        .dark { background-color: #b58863; }
        .selected { background-color: rgba(103, 255, 103, 0.6) !important; }
        .possible-move { background-color: rgba(255, 255, 103, 0.6) !important; }
        .possible-capture { background-color: rgba(255, 103, 103, 0.6) !important; }
        .controls { display: flex; gap: 15px; justify-content: center; margin-top: 20px; }
        button {
            padding: 10px 20px;
            background: linear-gradient(45deg, #ff6b6b, #ff8e8e);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
        }
        h1 { text-align: center; margin-bottom: 20px; }
        .player-info { display: flex; justify-content: space-between; margin: 10px 0; }
        .connection-status { text-align: center; margin: 10px 0; }
    </style>
    <!-- Add PeerJS for multiplayer -->
    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
</head>
<body>
    <div style="text-align: center;">
        <h1>شەترەنجی هاوڕێکان</h1>
        <div class="player-info">
            <div id="whitePlayer">سپی: تۆ</div>
            <div id="blackPlayer">ڕەش: چاوەڕوانی هاوڕێ</div>
        </div>
        <div class="chessboard" id="chessboard"></div>
        <div class="connection-status" id="connectionStatus">پەیوەندی نەکراوە</div>
        <div class="controls">
            <button id="copyCode">کۆپی کۆد</button>
            <button id="joinGame">بچۆ ناو یاریەکە</button>
        </div>
        <div>
            <input type="text" id="gameCodeInput" placeholder="کۆدی هاوڕێ">
        </div>
    </div>

    <script>
        // Game setup
        const chessboard = document.getElementById('chessboard');
        const whitePlayer = document.getElementById('whitePlayer');
        const blackPlayer = document.getElementById('blackPlayer');
        const connectionStatus = document.getElementById('connectionStatus');
        const copyCodeBtn = document.getElementById('copyCode');
        const joinGameBtn = document.getElementById('joinGame');
        const gameCodeInput = document.getElementById('gameCodeInput');

        // Chess board state
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
        let selectedSquare = null;
        let possibleMoves = [];
        let currentTurn = 'white';
        let playerColor = 'white';
        let peer = null;
        let conn = null;

        // Initialize PeerJS
        function initPeerJS() {
            peer = new Peer();

            peer.on('open', (id) => {
                connectionStatus.textContent = `کۆدی تۆ: ${id}`;
                gameCodeInput.placeholder = id;
            });

            peer.on('connection', (connection) => {
                conn = connection;
                playerColor = 'white';
                connectionStatus.textContent = "هاوڕێ پەیوەندی کرد!";
                
                conn.on('data', (data) => {
                    if (data.type === 'move') {
                        applyMove(data.from, data.to);
                    }
                });
            });
        }

        // Join a game
        function joinGame() {
            const friendCode = gameCodeInput.value.trim();
            if (!friendCode) return;

            conn = peer.connect(friendCode);
            playerColor = 'black';

            conn.on('open', () => {
                connectionStatus.textContent = "پەیوەندی کراوە";
            });

            conn.on('data', (data) => {
                if (data.type === 'move') {
                    applyMove(data.from, data.to);
                }
            });
        }

        // Apply opponent's move
        function applyMove(from, to) {
            currentBoard[to.row][to.col] = currentBoard[from.row][from.col];
            currentBoard[from.row][from.col] = '';
            currentTurn = 'white';
            createBoard();
        }

        // Create chessboard UI
        function createBoard() {
            chessboard.innerHTML = '';
            for (let row = 0; row < 8; row++) {
                for (let col = 0; col < 8; col++) {
                    const square = document.createElement('div');
                    square.className = `square ${(row + col) % 2 === 0 ? 'light' : 'dark'}`;
                    square.textContent = currentBoard[row][col];
                    square.dataset.row = row;
                    square.dataset.col = col;

                    if (selectedSquare && selectedSquare.row === row && selectedSquare.col === col) {
                        square.classList.add('selected');
                    }

                    square.addEventListener('click', () => handleSquareClick(row, col));
                    chessboard.appendChild(square);
                }
            }

            // Update player info
            whitePlayer.textContent = playerColor === 'white' ? 'سپی: تۆ' : 'سپی: هاوڕێ';
            blackPlayer.textContent = playerColor === 'black' ? 'ڕەش: تۆ' : 'ڕەش: هاوڕێ';
        }

        // Handle square clicks
        function handleSquareClick(row, col) {
            if (currentTurn !== playerColor) return;

            const piece = currentBoard[row][col];

            if (!selectedSquare && piece && isCurrentTurnPiece(piece)) {
                selectedSquare = { row, col };
                possibleMoves = getPossibleMoves(row, col, piece);
                createBoard();
                return;
            }

            if (selectedSquare) {
                const isMoveValid = possibleMoves.some(move => 
                    move.row === row && move.col === col
                );

                if (isMoveValid) {
                    makeMove(selectedSquare.row, selectedSquare.col, row, col);
                }

                selectedSquare = null;
                possibleMoves = [];
                createBoard();
            }
        }

        // Make a move and send to opponent
        function makeMove(fromRow, fromCol, toRow, toCol) {
            currentBoard[toRow][toCol] = currentBoard[fromRow][fromCol];
            currentBoard[fromRow][fromCol] = '';
            currentTurn = 'black';

            if (conn && conn.open) {
                conn.send({
                    type: 'move',
                    from: { row: fromRow, col: fromCol },
                    to: { row: toRow, col: toCol }
                });
            }

            createBoard();
        }

        // Helper functions
        function isCurrentTurnPiece(piece) {
            const isWhitePiece = '♙♖♘♗♕♔'.includes(piece);
            return (currentTurn === 'white' && isWhitePiece) || (currentTurn === 'black' && !isWhitePiece);
        }

        function getPossibleMoves(row, col, piece) {
            // Simplified move logic (add full chess rules if needed)
            const moves = [];
            const isWhite = '♙♖♘♗♕♔'.includes(piece);

            if (piece === '♙' || piece === '♟') {
                const direction = isWhite ? -1 : 1;
                if (isEmpty(row + direction, col)) {
                    moves.push({ row: row + direction, col });
                }
            }

            return moves;
        }

        function isEmpty(row, col) {
            return row >= 0 && row < 8 && col >= 0 && col < 8 && currentBoard[row][col] === '';
        }

        // Initialize the game
        initPeerJS();
        createBoard();

        // Event listeners
        copyCodeBtn.addEventListener('click', () => {
            navigator.clipboard.writeText(gameCodeInput.placeholder);
            alert('کۆدی یاری کۆپی کراوە!');
        });

        joinGameBtn.addEventListener('click', joinGame);
    </script>
</body>
</html>

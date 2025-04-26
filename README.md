 <head>
    <!-- Add these lines -->
    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/peerjs@1.4.7/dist/peerjs.min.js"></script>
</head>

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
let opponentName = "Opponent";
let whiteTime = 600;
let blackTime = 600;
let timerInterval = null;

// PeerJS Connection
let peer = null;
let conn = null;

// Initialize
function init() {
    setupPeerJS();
    setupEventListeners();
    createBoard();
    startTimer();
}

// PeerJS Setup
function setupPeerJS() {
    peer = new Peer();
    
    peer.on('open', (id) => {
        friendCode.value = id;
        connectionStatus.textContent = "ئامادە بۆ پەیوەندی";
        connectionStatus.style.color = "#4CAF50";
    });
    
    peer.on('connection', (connection) => {
        conn = connection;
        conn.on('open', () => {
            opponentName = "هاوڕێ";
            playerColor = 'white';
            connectionStatus.textContent = "هاوڕێ پەیوەندی کرد";
            updatePlayerInfo();
            createBoard();
            addChatMessage("سیستەم", "هاوڕێ پەیوەندی کرد!");
        });
        
        conn.on('data', handleIncomingData);
        conn.on('close', () => {
            connectionStatus.textContent = "هاوڕێ جێهێشت";
            addChatMessage("سیستەم", "هاوڕێ جێهێشت!");
        });
    });
    
    peer.on('error', (err) => {
        console.error(err);
        connectionStatus.textContent = "هەڵە: " + err.type;
        connectionStatus.style.color = "#F44336";
    });
}

function handleIncomingData(data) {
    switch(data.type) {
        case 'move':
            applyOpponentMove(data.from, data.to);
            break;
        case 'chat':
            addChatMessage(opponentName, data.message);
            break;
        case 'draw':
            handleDrawOffer();
            break;
        case 'reset':
            handleGameReset();
            break;
    }
}

// Game Functions
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

            if (selectedSquare && selectedSquare.row == displayRow && selectedSquare.col == displayCol) {
                square.classList.add('selected');
            }

            if (currentTurn === playerColor) {
                square.addEventListener('click', () => handleSquareClick(displayRow, displayCol));
            } else {
                square.style.cursor = 'not-allowed';
            }

            chessboard.appendChild(square);
        }
    }

    updatePlayerInfo();
    updateGameStatus();
}

function handleSquareClick(row, col) {
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

function makeMove(fromRow, fromCol, toRow, toCol) {
    const pieceMoved = currentBoard[fromRow][fromCol];
    currentBoard[toRow][toCol] = pieceMoved;
    currentBoard[fromRow][fromCol] = '';
    
    // Pawn promotion
    if ((pieceMoved === '♙' && toRow === 0) || (pieceMoved === '♟' && toRow === 7)) {
        currentBoard[toRow][toCol] = pieceMoved === '♙' ? '♕' : '♛';
    }
    
    currentTurn = currentTurn === 'white' ? 'black' : 'white';
    
    if (conn && conn.open) {
        conn.send({
            type: 'move',
            from: { row: fromRow, col: fromCol },
            to: { row: toRow, col: toCol }
        });
    }
}

function applyOpponentMove(from, to) {
    currentBoard[to.row][to.col] = currentBoard[from.row][from.col];
    currentBoard[from.row][from.col] = '';
    currentTurn = playerColor; // Switch turn back to you
    createBoard();
}

// Multiplayer Functions
function joinGame() {
    const code = joinCode.value.trim();
    if (!code) return;
    
    conn = peer.connect(code);
    
    conn.on('open', () => {
        gameId = code;
        playerColor = 'black';
        opponentName = "Host";
        isFlipped = true;
        connectionStatus.textContent = "پەیوەندی کراوە";
        updatePlayerInfo();
        createBoard();
        addChatMessage("سیستەم", `پەیوەندی بە یاری ${code} کرد!`);
    });
    
    conn.on('data', handleIncomingData);
    conn.on('close', () => {
        connectionStatus.textContent = "هاوڕێ جێهێشت";
    });
}

function sendChatMessage() {
    const message = chatInput.value.trim();
    if (message) {
        addChatMessage("تۆ", message);
        if (conn && conn.open) {
            conn.send({ type: 'chat', message });
        }
        chatInput.value = '';
    }
}

// Helper Functions
function addChatMessage(sender, message) {
    const messageDiv = document.createElement('div');
    messageDiv.className = `chat-message ${sender === "تۆ" ? 'my-message' : 'friend-message'}`;
    messageDiv.innerHTML = `<strong>${sender}:</strong> ${message}`;
    chatBox.appendChild(messageDiv);
    chatBox.scrollTop = chatBox.scrollHeight;
}

function updatePlayerInfo() {
    if (playerColor === 'white') {
        whitePlayer.textContent = `سپی: تۆ`;
        blackPlayer.textContent = `ڕەش: ${opponentName}`;
    } else {
        whitePlayer.textContent = `سپی: ${opponentName}`;
        blackPlayer.textContent = `ڕەش: تۆ`;
    }
}

// Initialize the game
init();

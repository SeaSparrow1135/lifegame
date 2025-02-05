<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ライフゲーム</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f4f4f4;
            margin: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .controls {
            margin-bottom: 10px;
        }
        .grid-container {
            display: flex;
            justify-content: center;
        }
        .grid {
            display: grid;
            gap: 1px;
        }
        .cell {
            width: 10px;
            height: 10px;
            background-color: white;
            border: 1px solid #ccc;
            transition: background-color 0.1s;
        }
        .alive {
            background-color: black;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            border: none;
            cursor: pointer;
            font-size: 16px;
            background-color: #007BFF;
            color: white;
            border-radius: 5px;
        }
        button:hover {
            background-color: #0056b3;
        }
        input {
            padding: 5px;
            font-size: 16px;
            width: 60px;
        }
        h1 {
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <h1>ライフゲーム</h1> <!-- ここでタイトルを追加 -->
    <div class="controls">
        <label for="rows">行数:</label>
        <input type="number" id="rows" value="64" min="1" max="1024">
        <label for="cols">列数:</label>
        <input type="number" id="cols" value="64" min="1" max="1024">
        <button onclick="setGridSize()">適用</button>
    </div>
    <div class="grid-container">
        <div class="grid" id="grid"></div>
    </div>
    <div class="controls">
        <button onclick="toggleGame()" id="toggleButton">開始</button>
        <button onclick="resetGame()">リセット</button>
        <button onclick="undoGame()">1コマ戻す</button>
        <button onclick="updateGame()">1コマ進める</button>
    </div>
    <script>
        let rows = 64;
        let cols = 64;
        let grid = [];
        let history = [];
        let running = false;
        let interval;
        let mouseDown = false;
        const gridElement = document.getElementById("grid");
        const toggleButton = document.getElementById("toggleButton");
        
        document.addEventListener("mousedown", () => mouseDown = true);
        document.addEventListener("mouseup", () => mouseDown = false);

        function setGridSize() {
            rows = Math.min(1024, Math.max(1, document.getElementById("rows").value));
            cols = Math.min(1024, Math.max(1, document.getElementById("cols").value));
            resetGame();
        }

        function createGrid() {
            grid = Array.from({ length: rows }, () => Array(cols).fill(0));
            gridElement.style.gridTemplateColumns = `repeat(${cols}, 10px)`;
            gridElement.style.gridTemplateRows = `repeat(${rows}, 10px)`;
            gridElement.innerHTML = "";
            for (let row = 0; row < rows; row++) {
                for (let col = 0; col < cols; col++) {
                    const cell = document.createElement("div");
                    cell.classList.add("cell");
                    cell.dataset.row = row;
                    cell.dataset.col = col;
                    cell.addEventListener("mousedown", toggleCell);
                    cell.addEventListener("mouseover", drawCell);
                    gridElement.appendChild(cell);
                }
            }
        }

        function toggleCell(event) {
            const row = event.target.dataset.row;
            const col = event.target.dataset.col;
            grid[row][col] = grid[row][col] ? 0 : 1;
            updateGrid();
        }

        function drawCell(event) {
            if (mouseDown) {
                toggleCell(event);
            }
        }

        function updateGrid() {
            document.querySelectorAll(".cell").forEach(cell => {
                const row = cell.dataset.row;
                const col = cell.dataset.col;
                cell.classList.toggle("alive", grid[row][col] === 1);
            });
        }

        function toggleGame() {
            running = !running;
            toggleButton.textContent = running ? "停止" : "開始";
            if (running) {
                interval = setInterval(updateGame, 100);
            } else {
                clearInterval(interval);
            }
        }

        function updateGame() {
            let newGrid = grid.map(arr => [...arr]);
            for (let r = 0; r < rows; r++) {
                for (let c = 0; c < cols; c++) {
                    let neighbors = 0;
                    for (let dr of [-1, 0, 1]) {
                        for (let dc of [-1, 0, 1]) {
                            if (dr !== 0 || dc !== 0) {
                                let nr = r + dr, nc = c + dc;
                                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                                    neighbors += grid[nr][nc];
                                }
                            }
                        }
                    }
                    if (grid[r][c] === 1 && (neighbors < 2 || neighbors > 3)) newGrid[r][c] = 0;
                    if (grid[r][c] === 0 && neighbors === 3) newGrid[r][c] = 1;
                }
            }
            history.push(grid.map(arr => [...arr]));
            grid = newGrid;
            updateGrid();
        }

        function undoGame() {
            if (history.length > 0) {
                grid = history.pop();
                updateGrid();
            }
        }

        function resetGame() {
            running = false;
            clearInterval(interval);
            toggleButton.textContent = "開始";
            history = [];
            createGrid();
            updateGrid();
        }

        createGrid();
    </script>
</body>
</html>

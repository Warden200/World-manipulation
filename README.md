<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>World manipulation</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1a1a1a;
            color: #ffffff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            user-select: none;
        }

        #app-container {
            display: flex;
            width: 100vw;
            height: 100vw;
            max-height: 100vh;
        }

        #canvas-wrapper {
            position: relative;
            flex-grow: 1;
            height: 100%;
            background-color: #0b1d33;
            overflow: hidden;
            cursor: grab;
        }

        #canvas-wrapper.drawing-mode {
            cursor: crosshair;
        }

        #canvas-wrapper:active {
            cursor: grabbing;
        }

        #canvas-wrapper.drawing-mode:active {
            cursor: crosshair;
        }

        canvas {
            position: absolute;
            image-rendering: pixelated;
            background-color: #222;
            transform-origin: 0 0;
        }

        /* Боковая панель */
        #sidebar {
            width: 320px;
            background-color: #252526;
            border-left: 2px solid #3e3e42;
            display: flex;
            flex-direction: column;
            box-sizing: border-box;
            z-index: 10;
            overflow-y: auto;
        }

        .panel-section {
            padding: 15px;
            border-bottom: 1px solid #3e3e42;
        }

        h3 {
            margin-top: 0;
            margin-bottom: 12px;
            font-size: 16px;
            color: #007acc;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .btn {
            display: block;
            width: 100%;
            background-color: #333337;
            color: #fff;
            border: 1px solid #555;
            padding: 8px 12px;
            margin-bottom: 8px;
            border-radius: 4px;
            cursor: pointer;
            text-align: left;
            font-size: 14px;
            transition: background 0.2s;
        }

        .btn:hover {
            background-color: #3f3f46;
        }

        .btn.active {
            background-color: #007acc;
            border-color: #0098ff;
        }

        .btn-danger {
            background-color: #8a2525;
        }
        .btn-danger:hover {
            background-color: #a63232;
        }

        .control-group {
            margin-bottom: 10px;
        }

        label {
            display: block;
            font-size: 12px;
            color: #aaa;
            margin-bottom: 4px;
        }

        input[type="range"], input[type="text"], input[type="color"] {
            width: 100%;
            box-sizing: border-box;
            background: #1e1e1e;
            border: 1px solid #555;
            color: #fff;
            padding: 4px;
            border-radius: 4px;
        }

        /* Индикатор Войны (Верхний оверлей) */
        #war-overlay {
            position: absolute;
            top: 15px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.85);
            padding: 10px 20px;
            border-radius: 8px;
            border: 1px solid #555;
            display: flex;
            align-items: center;
            gap: 15px;
            pointer-events: none;
            z-index: 5;
            font-weight: bold;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            opacity: 0;
            transition: opacity 0.3s;
        }

        #war-overlay.visible {
            opacity: 1;
        }

        .overlay-team {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .flag-icon {
            width: 24px;
            height: 16px;
            display: inline-flex;
            flex-direction: column;
            border: 1px solid #fff;
        }

        .flag-stripe {
            flex: 1;
            width: 100%;
        }

        /* Лог событий */
        #log-container {
            flex-grow: 1;
            padding: 10px;
            font-family: monospace;
            font-size: 12px;
            overflow-y: auto;
            background-color: #1e1e1e;
            color: #888;
            max-height: 200px;
        }

        .log-entry {
            margin-bottom: 4px;
            line-height: 1.4;
        }

        /* Модалка редактирования */
        #edit-modal {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: #252526;
            border: 2px solid #007acc;
            border-radius: 8px;
            padding: 20px;
            z-index: 100;
            width: 280px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.7);
        }

        .modal-buttons {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }

        .flag-preview-box {
            display: flex;
            flex-direction: column;
            width: 60px;
            height: 40px;
            border: 1px solid #fff;
            margin: 10px auto;
        }

        /* Плашка текущего режима рисования */
        #draw-hint {
            display: none;
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(138, 37, 37, 0.9);
            padding: 10px 20px;
            border-radius: 20px;
            border: 1px solid #ff5555;
            z-index: 6;
            align-items: center;
            gap: 15px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }

        #draw-hint button {
            background: #fff;
            color: #000;
            border: none;
            padding: 2px 8px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
        }

        /* Плашка Паузы */
        #pause-indicator {
            display: none;
            position: absolute;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(200, 140, 0, 0.9);
            color: #fff;
            padding: 6px 16px;
            border-radius: 4px;
            font-weight: bold;
            font-size: 14px;
            letter-spacing: 1px;
            z-index: 4;
            box-shadow: 0 2px 8px rgba(0,0,0,0.4);
            pointer-events: none;
        }
    </style>
</head>
<body>

<div id="app-container">
    <div id="canvas-wrapper">
        <!-- Индикатор Паузы -->
        <div id="pause-indicator">⏸ СИМУЛЯЦИЯ НА ПАУЗЕ</div>

        <!-- Оверлей Войны -->
        <div id="war-overlay">
            <div class="overlay-team" id="war-team-a">
                <div class="flag-icon" id="overlay-flag-a"></div>
                <span id="overlay-name-a">Нация А</span>
                <span id="overlay-power-a">(0)</span>
            </div>
            <div style="color: #ff5555; font-size: 20px;">⚔️</div>
            <div class="overlay-team" id="war-team-b">
                <span id="overlay-power-b">(0)</span>
                <span id="overlay-name-b">Нация Б</span>
                <div class="flag-icon" id="overlay-flag-b"></div>
            </div>
        </div>

        <!-- Индикатор режима рисования -->
        <div id="draw-hint">
            <span id="draw-hint-text">Режим рисования: выберите страну</span>
            <button id="btn-close-draw">✕ Выйти</button>
        </div>

        <canvas id="gameCanvas" width="600" height="600"></canvas>
    </div>

    <!-- Боковая панель управления -->
    <div id="sidebar">
        <div class="panel-section">
            <h3>⚙️ Управление симуляцией</h3>
            <button class="btn btn-danger" id="btn-pause">⏸ Пауза</button>
            <div class="control-group" style="margin-top:10px;">
                <label id="speed-label">Скорость: 1x</label>
                <input type="range" id="input-speed" min="0" max="5" step="1" value="1">
            </div>
            <div style="font-size:11px; color:#aaa; text-align:center;">Удерживайте ПКМ / зажмите колесико для перемещения. Скролл для зума.</div>
        </div>

        <div class="panel-section">
            <h3>🛠️ Режим Бога</h3>
            <button class="btn" id="mode-draw">🖌️ Дорисовать пространство</button>
            <button class="btn" id="mode-create">🆕 Создать нацию</button>
            <button class="btn" id="mode-edit">✏️ Изменить нацию (Клик по стране)</button>
            <button class="btn" id="mode-war">⚔️ Объявить войну</button>
            <button class="btn" id="mode-peace">🕊️ Заключить мир</button>
            <button class="btn" id="mode-revolt">🔥 Вызвать революцию</button>
            
            <div class="control-group" id="brush-size-group" style="display:none; margin-top:8px;">
                <label id="brush-label">Размер кисти: 3</label>
                <input type="range" id="input-brush" min="1" max="10" step="1" value="3">
            </div>
        </div>

        <div class="panel-section" style="flex-grow: 1; display:flex; flex-direction:column; min-height:150px;">
            <h3>📋 Лог событий</h3>
            <div id="log-container"></div>
        </div>
    </div>
</div>

<!-- Модальное окно кастомизации -->
<div id="edit-modal">
    <h3 style="margin-bottom:15px; text-align:center;">Редактор Нации</h3>
    <div class="control-group">
        <label>Название страны:</label>
        <input type="text" id="edit-name">
    </div>
    <div class="control-group">
        <label>Основной цвет на карте:</label>
        <input type="color" id="edit-color">
    </div>
    <div class="control-group">
        <label>Цвета флага (Верх / Середина / Низ):</label>
        <div style="display:flex; gap:4px;">
            <input type="color" id="edit-f1" style="padding:0; height:25px;">
            <input type="color" id="edit-f2" style="padding:0; height:25px;">
            <input type="color" id="edit-f3" style="padding:0; height:25px;">
        </div>
        <div class="flag-preview-box" id="edit-flag-preview">
            <div class="flag-stripe" id="preview-s1"></div>
            <div class="flag-stripe" id="preview-s2"></div>
            <div class="flag-stripe" id="preview-s3"></div>
        </div>
    </div>
    <div class="modal-buttons">
        <button class="btn" id="btn-modal-save" style="background:#007acc; margin:0;">Сохранить</button>
        <button class="btn" id="btn-modal-cancel" style="margin:0;">Отмена</button>
    </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const wrapper = document.getElementById('canvas-wrapper');

// Параметры карты и сетки
const MAP_SIZE = 600;
canvas.width = MAP_SIZE;
canvas.height = MAP_SIZE;

// Сетка пикселей земли. 0 = пустая земля, >0 = ID фракции
const grid = new Uint16Array(MAP_SIZE * MAP_SIZE);

// Состояние камеры (Зум и перемещение)
let zoom = 1.0;
let offsetX = 0;
let offsetY = 0;
let isPanning = false;
let startX, startY;

// Режимы игры
let currentMode = null; // 'draw', 'create', 'edit', 'war', 'peace', 'revolt'
let isDrawing = false;
let selectedNationForDrawing = null;
let brushSize = 3;

// Войны и нации
let isPaused = false;
let simSpeed = 1;
let nations = {};
let nextNationId = 1;
let activeWars = []; // массив объектов {a: id, b: id, timer: frames}

// Наведение/логирование для интерфейса объявлений
let firstSelectedId = null;

// Инициализация камеры по центру
function centerCamera() {
    const wRect = wrapper.getBoundingClientRect();
    offsetX = (wRect.width - MAP_SIZE * zoom) / 2;
    offsetY = (wRect.height - MAP_SIZE * zoom) / 2;
    updateTransform();
}

function updateTransform() {
    canvas.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${zoom})`;
}

// Генерация стандартной квадратной карты "Куб" (Суша по центру, океан по краям)
function initCubeMap() {
    grid.fill(0); // Очищаем всё
    // Заполняем центр сушей, края оставляем океаном (обозначим океан внутренней логикой или просто пустым)
    // В Ages of Conflict "Куб" - это сплошной квадрат суши, окруженный границами.
    // Наполним базовыми случайными нациями
    const initialNationsCount = 12;
    const names = ["Империя", "Республика", "Княжество", "Королевство", "Союз", "Федерация", "Орда", "Гетманат", "Штат", "Директория", "Каганат", "Султанат"];
    const prefixes = ["Северный", "Южный", "Новый", "Великий", "Западный", "Восточный", "Вольный", "Священный"];

    for (let i = 0; i < initialNationsCount; i++) {
        let id = nextNationId++;
        let name = prefixes[Math.floor(Math.random() * prefixes.length)] + " " + names[Math.floor(Math.random() * names.length)] + " " + id;
        let color = getRandomColor();
        let f1 = getRandomColor();
        let f2 = getRandomColor();
        let f3 = getRandomColor();
        
        nations[id] = {
            id: id,
            name: name,
            color: color,
            flag: [f1, f2, f3],
            population: 100,
            budget: 1000,
            pixels: 0,
            isAlive: true
        };

        // Размещаем начальные столицы подальше от краев
        let rx = Math.floor(50 + Math.random() * (MAP_SIZE - 100));
        let ry = Math.floor(50 + Math.random() * (MAP_SIZE - 100));
        
        // Маленькое ядро 5x5
        for(let dy = -3; dy <= 3; dy++) {
            for(let dx = -3; dx <= 3; dx++) {
                let px = rx + dx;
                let py = ry + dy;
                if(px >= 0 && px < MAP_SIZE && py >= 0 && py < MAP_SIZE) {
                    grid[py * MAP_SIZE + px] = id;
                }
            }
        }
    }
    recalculatePixelCounts();
    logEvent("🗺️ Карта Куб успешно создана! Размещено 12 наций.");
}

function getRandomColor() {
    const letters = '0123456789ABCDEF';
    let color = '#';
    for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
}

function recalculatePixelCounts() {
    for (let id in nations) nations[id].pixels = 0;
    for (let i = 0; i < grid.length; i++) {
        let id = grid[i];
        if (id > 0 && nations[id]) {
            nations[id].pixels++;
        }
    }
}

// Преобразование координат экрана в координаты игрового Canvas пикселя
function getGridCoords(e) {
    const rect = canvas.getBoundingClientRect();
    const x = Math.floor((e.clientX - rect.left) / zoom);
    const y = Math.floor((e.clientY - rect.top) / zoom);
    return { x, y };
}

// Рендеринг флага в HTML строку
function makeFlagHTML(colors) {
    return `<div class="flag-stripe" style="background:${colors[0]}"></div>
            <div class="flag-stripe" style="background:${colors[1]}"></div>
            <div class="flag-stripe" style="background:${colors[2]}"></div>`;
}

function logEvent(text) {
    const logContainer = document.getElementById('log-container');
    const entry = document.createElement('div');
    entry.className = 'log-entry';
    entry.innerText = text;
    logContainer.appendChild(entry);
    logContainer.scrollTop = logContainer.scrollHeight;
}

// Логика симуляции (Один шаг расширения / войны)
function simulationStep() {
    if (isPaused) return;

    // 1. Экономика и население
    for (let id in nations) {
        let n = nations[id];
        if (!n.isAlive) continue;
        
        if (n.pixels === 0) {
            n.isAlive = false;
            logEvent(`💀 Нация "${n.name}" была полностью уничтожена!`);
            // Прекращаем её войны
            activeWars = activeWars.filter(w => w.a !== n.id && w.b !== n.id);
            continue;
        }

        // Прирост населения от территории
        n.population += Math.floor(n.pixels * 0.05) + 1;
        
        // Проверяем состояние войны
        let inWar = activeWars.some(w => w.a === n.id || w.b === n.id);
        if (inWar) {
            n.budget -= Math.floor(n.population * 0.02) + 5; // Траты на войну
        } else {
            n.budget += Math.floor(n.pixels * 0.1) + 2; // Мирные налоги
        }

        if (n.budget < 0) n.budget = 0;
    }

    // Случайные объявления войн между соседями
    if (Math.random() < 0.015 && activeWars.length < 5) {
        triggerRandomWar();
    }

    // 2. Расширение пикселей и боевые действия
    // Делаем выборку случайных пикселей для симуляции фронта (оптимизация)
    const checks = Math.floor(MAP_SIZE * MAP_SIZE * 0.08);
    for (let k = 0; k < checks; k++) {
        let x = Math.floor(Math.random() * MAP_SIZE);
        let y = Math.floor(Math.random() * MAP_SIZE);
        let idx = y * MAP_SIZE + x;
        let currentOwner = grid[idx];

        if (currentOwner > 0) {
            // Пытаемся захватить соседа
            let dir = Math.floor(Math.random() * 4);
            let nx = x, ny = y;
            if (dir === 0) nx++;
            else if (dir === 1) nx--;
            else if (dir === 2) ny++;
            else ny--;

            if (nx >= 0 && nx < MAP_SIZE && ny >= 0 && ny < MAP_SIZE) {
                let nIdx = ny * MAP_SIZE + nx;
                let targetOwner = grid[nIdx];

                if (targetOwner === 0) {
                    // Экспансия на пустую землю
                    grid[nIdx] = currentOwner;
                    nations[currentOwner].pixels++;
                } else if (targetOwner !== currentOwner) {
                    // Проверяем, воюют ли они
                    let war = activeWars.find(w => (w.a === currentOwner && w.b === targetOwner) || (w.a === targetOwner && w.b === currentOwner));
                    if (war) {
                        let attacker = nations[currentOwner];
                        let defender = nations[targetOwner];

                        // Сила атаки падает, если нет бюджета
                        let attackPower = attacker.budget > 0 ? Math.random() * 1.3 : Math.random() * 0.6;
                        let defensePower = defender.budget > 0 ? Math.random() * 1.0 : Math.random() * 0.5;

                        if (attackPower > defensePower) {
                            grid[nIdx] = currentOwner;
                            attacker.pixels++;
                            defender.pixels--;
                        }
                    }
                }
            }
        }
    }
}

function triggerRandomWar() {
    let keys = Object.keys(nations).filter(id => nations[id].isAlive);
    if (keys.length < 2) return;
    let a = keys[Math.floor(Math.random() * keys.length)];
    let b = keys[Math.floor(Math.random() * keys.length)];
    if (a !== b && !activeWars.some(w => (w.a === a && w.b === b) || (w.a === b && w.b === a))) {
        startWar(a, b);
    }
}

function startWar(idA, idB) {
    activeWars.push({ a: parseInt(idA), b: parseInt(idB) });
    logEvent(`⚔️ Война объявлена между "${nations[idA].name}" и "${nations[idB].name}"!`);
}

// Отрисовка карты и фронтовых флагов
let frontLinesCache = []; // кэшированные точки столкновения для отрисовки флагов

function drawGame() {
    // Очистка
    ctx.fillStyle = "#111";
    ctx.fillRect(0, 0, MAP_SIZE, MAP_SIZE);

    // Отрисовка пикселей наций
    // Для скорости создаем ImageData
    const imgData = ctx.createImageData(MAP_SIZE, MAP_SIZE);
    const data = imgData.data;

    // Подготовим hex-to-rgb маппинг для скорости
    const colorMap = {};
    for (let id in nations) {
        let c = nations[id].color;
        // Простой парсинг #hex
        let r = parseInt(c.substr(1,2), 16);
        let g = parseInt(c.substr(3,2), 16);
        let b = parseInt(c.substr(5,2), 16);
        colorMap[id] = {r, g, b};
    }

    frontLinesCache = [];

    for (let y = 0; y < MAP_SIZE; y++) {
        for (let x = 0; x < MAP_SIZE; x++) {
            let idx = y * MAP_SIZE + x;
            let id = grid[idx];
            let dIdx = idx * 4;

            if (id > 0 && nations[id] && nations[id].isAlive) {
                let rgb = colorMap[id];
                data[dIdx] = rgb.r;
                data[dIdx+1] = rgb.g;
                data[dIdx+2] = rgb.b;
                data[dIdx+3] = 255;

                // Быстрое определение границ и линий фронта войн (каждый 4-й пиксель для оптимизации)
                if (x % 3 === 0 && y % 3 === 0 && x < MAP_SIZE - 1 && y < MAP_SIZE - 1) {
                    let rightId = grid[idx + 1];
                    let downId = grid[idx + MAP_SIZE];
                    
                    if (rightId > 0 && rightId !== id) {
                        checkFrontline(id, rightId, x, y);
                    }
                    if (downId > 0 && downId !== id) {
                        checkFrontline(id, downId, x, y);
                    }
                }
            } else {
                // Пустая земля / Куб фон
                data[dIdx] = 34;
                data[dIdx+1] = 34;
                data[dIdx+2] = 34;
                data[dIdx+3] = 255;
            }
        }
    }
    ctx.putImageData(imgData, 0, 0);

    // Отрисовка "Шагающих флагов" на линиях фронта (как в Ages of Conflict)
    drawFrontlineFlags();

    // Обновление верхнего оверлея войны
    updateWarOverlay();
}

function checkFrontline(idA, idB, x, y) {
    let war = activeWars.find(w => (w.a === idA && w.b === idB) || (w.a === idB && w.b === idA));
    if (war) {
        // Добавляем точку фронта, если в этой зоне еще нет флага (прореживание флагов)
        let tooClose = frontLinesCache.some(pt => Math.hypot(pt.x - x, pt.y - y) < 45);
        if (!tooClose) {
            frontLinesCache.push({ idA, idB, x, y });
        }
    }
}

function drawFrontlineFlags() {
    const time = Date.now() * 0.004;
    frontLinesCache.forEach(pt => {
        let nA = nations[pt.idA];
        let nB = nations[pt.idB];
        if (!nA || !nB) return;

        let wave = Math.sin(time + pt.x) * 3; // Анимация покачивания

        ctx.save();
        // Рисуем флаг стороны А (чуть левее/выше границы)
        drawMiniFlag(ctx, nA.flag, pt.x - 12, pt.y - 10 + wave);
        // Скрещенные мечи посередине
        ctx.fillStyle = "#ff3333";
        ctx.font = "bold 9px Arial";
        ctx.fillText("⚔️", pt.x - 4, pt.y + 4);
        // Рисуем флаг стороны Б (чуть правее/ниже границы)
        drawMiniFlag(ctx, nB.flag, pt.x + 8, pt.y - 10 - wave);
        ctx.restore();
    });
}

function drawMiniFlag(context, flagColors, x, y) {
    const w = 12;
    const h = 8;
    context.strokeStyle = "#ffffff";
    context.lineWidth = 1;
    context.strokeRect(x, y, w, h);
    
    context.fillStyle = flagColors[0];
    context.fillRect(x, y, w, 2.7);
    context.fillStyle = flagColors[1];
    context.fillRect(x, y + 2.7, w, 2.7);
    context.fillStyle = flagColors[2];
    context.fillRect(x, y + 5.4, w, 2.7);
}

function updateWarOverlay() {
    const overlay = document.getElementById('war-overlay');
    if (activeWars.length === 0) {
        overlay.className = '';
        return;
    }
    // Показываем самую активную / первую войну в списке
    let war = activeWars[0];
    let nA = nations[war.a];
    let nB = nations[war.b];

    if (!nA || !nB || !nA.isAlive || !nB.isAlive) {
        overlay.className = '';
        return;
    }

    overlay.className = 'visible';
    document.getElementById('overlay-name-a').innerText = nA.name;
    document.getElementById('overlay-power-a').innerText = `(${nA.pixels} px)`;
    document.getElementById('overlay-flag-a').innerHTML = makeFlagHTML(nA.flag);

    document.getElementById('overlay-name-b').innerText = nB.name;
    document.getElementById('overlay-power-b').innerText = `(${nB.pixels} px)`;
    document.getElementById('overlay-flag-b').innerHTML = makeFlagHTML(nB.flag);
}

// Главный цикл игры
function loop() {
    for (let s = 0; s < simSpeed; s++) {
        simulationStep();
    }
    drawGame();
    requestAnimationFrame(loop);
}

// Взаимодействие с картой (Мышь, зум, клики)
wrapper.addEventListener('wheel', (e) => {
    e.preventDefault();
    const zoomFactor = 1.1;
    let newZoom = e.deltaY < 0 ? zoom * zoomFactor : zoom / zoomFactor;
    if (newZoom < 0.5) newZoom = 0.5;
    if (newZoom > 15) newZoom = 15;

    // Зум в точку мыши
    const rect = canvas.getBoundingClientRect();
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;

    offsetX -= (mouseX / zoom) * (newZoom - zoom);
    offsetY -= (mouseY / zoom) * (newZoom - zoom);
    zoom = newZoom;

    updateTransform();
}, { passive: false });

// Перетаскивание карты (Удерживая ПКМ или Колесико)
wrapper.addEventListener('mousedown', (e) => {
    if (currentMode === 'draw' && e.button === 0) {
        isDrawing = true;
        handleDrawing(e);
        return;
    }

    if (e.button === 2 || e.button === 1) {
        isPanning = true;
        startX = e.clientX - offsetX;
        startY = e.clientY - offsetY;
    }
});

wrapper.addEventListener('mousemove', (e) => {
    if (isPanning) {
        offsetX = e.clientX - startX;
        offsetY = e.clientY - startY;
        updateTransform();
    } else if (isDrawing && currentMode === 'draw') {
        handleDrawing(e);
    }
});

window.addEventListener('mouseup', () => {
    isPanning = false;
    isDrawing = false;
});

// Отмена контекстного меню для ПКМ
wrapper.addEventListener('contextmenu', e => e.preventDefault());

// Логика кликов для Инструментов Бога
wrapper.addEventListener('click', (e) => {
    if (e.button !== 0 || currentMode === 'draw') return;

    const coords = getGridCoords(e);
    if (coords.x < 0 || coords.x >= MAP_SIZE || coords.y < 0 || coords.y >= MAP_SIZE) return;

    let clickedIdx = coords.y * MAP_SIZE + coords.x;
    let targetNationId = grid[clickedIdx];

    if (currentMode === 'create') {
        // Создание нации в пустом месте
        if (targetNationId === 0) {
            let id = nextNationId++;
            let name = "Новая Нация " + id;
            let color = getRandomColor();
            nations[id] = {
                id: id,
                name: name,
                color: color,
                flag: [getRandomColor(), getRandomColor(), getRandomColor()],
                population: 100,
                budget: 1000,
                pixels: 0,
                isAlive: true
            };
            // Рисуем круг
            for (let dy = -4; dy <= 4; dy++) {
                for (let dx = -4; dx <= 4; dx++) {
                    if (Math.hypot(dx, dy) <= 4) {
                        let px = coords.x + dx;
                        let py = coords.y + dy;
                        if (px >= 0 && px < MAP_SIZE && py >= 0 && py < MAP_SIZE) {
                            grid[py * MAP_SIZE + px] = id;
                        }
                    }
                }
            }
            recalculatePixelCounts();
            logEvent(`🆕 Господь создал нацию "${name}"!`);
            setMode(null);
        }
    } else if (currentMode === 'edit') {
        if (targetNationId > 0 && nations[targetNationId]) {
            openEditModal(targetNationId);
            setMode(null);
        }
    } else if (currentMode === 'war') {
        if (targetNationId > 0 && nations[targetNationId]) {
            if (!firstSelectedId) {
                firstSelectedId = targetNationId;
                logEvent(`👉 Выберите вторую нацию, чтобы объявить войну для "${nations[firstSelectedId].name}"`);
            } else {
                if (firstSelectedId !== targetNationId) {
                    startWar(firstSelectedId, targetNationId);
                    setMode(null);
                }
            }
        }
    } else if (currentMode === 'peace') {
        if (targetNationId > 0 && nations[targetNationId]) {
            if (!firstSelectedId) {
                firstSelectedId = targetNationId;
                logEvent(`👉 Выберите вторую нацию, чтобы заключить мир с "${nations[firstSelectedId].name}"`);
            } else {
                activeWars = activeWars.filter(w => !((w.a === firstSelectedId && w.b === targetNationId) || (w.a === targetNationId && w.b === firstSelectedId)));
                logEvent(`🕊️ Заключен мир между "${nations[firstSelectedId].name}" и "${nations[targetNationId].name}"`);
                setMode(null);
            }
        }
    } else if (currentMode === 'revolt') {
        if (targetNationId > 0 && nations[targetNationId]) {
            let parent = nations[targetNationId];
            let id = nextNationId++;
            let name = "Революционеры " + parent.name;
            nations[id] = {
                id: id,
                name: name,
                color: "#ff2222",
                flag: ["#ff0000", "#000000", "#cc0000"],
                population: Math.floor(parent.population * 0.3),
                budget: 500,
                pixels: 0,
                isAlive: true
            };
            // Перекрашиваем часть пикселей в округе клика под повстанцев
            for (let dy = -25; dy <= 25; dy++) {
                for (let dx = -25; dx <= 25; dx++) {
                    if (Math.hypot(dx, dy) <= 25) {
                        let px = coords.x + dx;
                        let py = coords.y + dy;
                        if (px >= 0 && px < MAP_SIZE && py >= 0 && py < MAP_SIZE) {
                            if (grid[py * MAP_SIZE + px] === parent.id) {
                                grid[py * MAP_SIZE + px] = id;
                            }
                        }
                    }
                }
            }
            recalculatePixelCounts();
            startWar(parent.id, id);
            logEvent(`🔥 Вспыхнула революция в "${parent.name}"! Созданы "${name}"`);
            setMode(null);
        }
    }
});

// Логика дорисовывания кистью пространства
function handleDrawing(e) {
    const coords = getGridCoords(e);
    if (coords.x < 0 || coords.x >= MAP_SIZE || coords.y < 0 || coords.y >= MAP_SIZE) return;

    if (!selectedNationForDrawing) {
        // Шаг 1: Кликнули по стране, чтобы взять ее в качестве активной кисти
        let clickedIdx = coords.y * MAP_SIZE + coords.x;
        let targetNationId = grid[clickedIdx];
        if (targetNationId > 0 && nations[targetNationId]) {
            selectedNationForDrawing = targetNationId;
            document.getElementById('draw-hint-text').innerText = `🖌️ Кисть: ${nations[selectedNationForDrawing].name}. Рисуйте по карте!`;
            logEvent(`🖌️ Инструмент кисти взят от нации "${nations[selectedNationForDrawing].name}". Зажмите ЛКМ и рисуйте.`);
        }
        isDrawing = false; // Сбрасываем флаг, чтобы начать рисовать только со следующего движения/нажатия
        return;
    }

    // Шаг 2: Рисование выбранной нацией
    let r = brushSize;
    for (let dy = -r; dy <= r; dy++) {
        for (let dx = -r; dx <= r; dx++) {
            if (dx*dx + dy*dy <= r*r) {
                let px = coords.x + dx;
                let py = coords.y + dy;
                if (px >= 0 && px < MAP_SIZE && py >= 0 && py < MAP_SIZE) {
                    grid[py * MAP_SIZE + px] = selectedNationForDrawing;
                }
            }
        }
    }
    recalculatePixelCounts();
}

// Установка текущего режима бога
function setMode(mode) {
    currentMode = mode;
    firstSelectedId = null;
    
    // Сбрасываем подсветку кнопок
    document.querySelectorAll('#sidebar .btn').forEach(b => {
        if(b.id !== 'btn-pause') b.classList.remove('active');
    });
    document.getElementById('brush-size-group').style.display = 'none';
    document.getElementById('draw-hint').style.display = 'none';
    wrapper.classList.remove('drawing-mode');

    if (!mode) return;

    // Подсвечиваем активный режим
    const btnMap = {
        'draw': 'mode-draw', 'create': 'mode-create', 'edit': 'mode-edit',
        'war': 'mode-war', 'peace': 'mode-peace', 'revolt': 'mode-revolt'
    };
    document.getElementById(btnMap[mode]).classList.add('active');

    if (mode === 'draw') {
        selectedNationForDrawing = null;
        document.getElementById('brush-size-group').style.display = 'block';
        document.getElementById('draw-hint').style.display = 'flex';
        document.getElementById('draw-hint-text').innerText = '🖌️ Шаг 1: Нажмите на страну, которой хотите дорисовать пространство';
        wrapper.classList.add('drawing-mode');
    } else {
        logEvent(`🔮 Активирован режим Бога: ${document.getElementById(btnMap[mode]).innerText}`);
    }
}

// Кнопки интерфейса режима Бога
document.getElementById('mode-draw').addEventListener('click', () => setMode('draw'));
document.getElementById('mode-create').addEventListener('click', () => setMode('create'));
document.getElementById('mode-edit').addEventListener('click', () => setMode('edit'));
document.getElementById('mode-war').addEventListener('click', () => setMode('war'));
document.getElementById('mode-peace').addEventListener('click', () => setMode('peace'));
document.getElementById('mode-revolt').addEventListener('click', () => setMode('revolt'));

document.getElementById('btn-close-draw').addEventListener('click', (e) => {
    e.stopPropagation();
    setMode(null);
});

// Кнопка Паузы
const pauseBtn = document.getElementById('btn-pause');
pauseBtn.addEventListener('click', () => {
    isPaused = !isPaused;
    if (isPaused) {
        pauseBtn.innerText = "▶️ Продолжить";
        pauseBtn.classList.remove('btn-danger');
        document.getElementById('pause-indicator').style.display = 'block';
        logEvent("⏸ Симуляция остановлена Богом.");
    } else {
        pauseBtn.innerText = "⏸ Пауза";
        pauseBtn.classList.add('btn-danger');
        document.getElementById('pause-indicator').style.display = 'none';
        logEvent("▶️ Симуляция продолжается.");
    }
});

// Настройка скорости
document.getElementById('input-speed').addEventListener('input', (e) => {
    simSpeed = parseInt(e.target.value);
    document.getElementById('speed-label').innerText = `Скорость: ${simSpeed}x`;
});

// Размер кисти рисования
document.getElementById('input-brush').addEventListener('input', (e) => {
    brushSize = parseInt(e.target.value);
    document.getElementById('brush-label').innerText = `Размер кисти: ${brushSize}`;
});

// Модальное окно Кастомизации
let currentEditingId = null;
function openEditModal(id) {
    let n = nations[id];
    currentEditingId = id;
    document.getElementById('edit-name').value = n.name;
    document.getElementById('edit-color').value = n.color;
    document.getElementById('edit-f1').value = n.flag[0];
    document.getElementById('edit-f2').value = n.flag[1];
    document.getElementById('edit-f3').value = n.flag[2];
    updateModalFlagPreview();
    document.getElementById('edit-modal').style.display = 'block';
}

function updateModalFlagPreview() {
    document.getElementById('preview-s1').style.backgroundColor = document.getElementById('edit-f1').value;
    document.getElementById('preview-s2').style.backgroundColor = document.getElementById('edit-f2').value;
    document.getElementById('preview-s3').style.backgroundColor = document.getElementById('edit-f3').value;
}

['edit-f1', 'edit-f2', 'edit-f3'].forEach(id => {
    document.getElementById(id).addEventListener('input', updateModalFlagPreview);
});

document.getElementById('btn-modal-save').addEventListener('click', () => {
    if (currentEditingId && nations[currentEditingId]) {
        let n = nations[currentEditingId];
        let oldName = n.name;
        n.name = document.getElementById('edit-name').value;
        n.color = document.getElementById('edit-color').value;
        n.flag = [
            document.getElementById('edit-f1').value,
            document.getElementById('edit-f2').value,
            document.getElementById('edit-f3').value
        ];
        logEvent(`✏️ Нация "${oldName}" переименована в "${n.name}" и полностью кастомизирована.`);
    }
    document.getElementById('edit-modal').style.display = 'none';
});

document.getElementById('btn-modal-cancel').addEventListener('click', () => {
    document.getElementById('edit-modal').style.display = 'none';
});

// Инициализация при старте
initCubeMap();
centerCamera();
loop();
</script>

</body>
</html>

<html lang="ru">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Mines — Animated Grid</title>
<style>
:root{
  --cell-size:56px;
  --gap:8px;
  --accent:#06b6d4;
  --text-light:#f8fafc;
  --grid-bg:#111; /* яркий черный фон контейнера */
  --cell-closed:#222; /* чуть темнее чем фон контейнера */
  --cell-highlight:#FFD700; /* цвет рамки при открытии */
  --cell-open-bg:#333; /* внутри открытой клетки немного светлее */
}

body{
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  display:flex;
  align-items:center;
  justify-content:center;
  min-height:100vh;
  margin:0;
  background: var(--grid-bg);
  color: var(--text-light);
}

.container{
  display:flex;
  flex-direction:column;
  align-items:center;
  gap:14px;
}

.number-control{
  display:flex;
  align-items:center;
  justify-content:center;
  gap:10px;
  margin-bottom:10px;
}

.num-box{
  width: calc(var(--cell-size) * 2);
  height: var(--cell-size);
  background: var(--cell-closed);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  font-weight: 700;
  color: var(--text-light);
  box-shadow: inset 0 -4px 0 rgba(0,0,0,0.3);
  user-select: none;
}

.pm-btn{
  width: 42px;
  height: 42px;
  border-radius: 50%;
  border: none;
  background: var(--accent);
  color: #022033;
  font-size: 24px;
  font-weight: 800;
  cursor: pointer;
  box-shadow: 0 6px 14px rgba(6,182,212,0.35);
  transition: transform .1s ease, box-shadow .2s;
}
.pm-btn:hover{ transform: scale(1.1); box-shadow:0 10px 20px rgba(6,182,212,0.45);}
.pm-btn:active{ transform: scale(0.95); }

.frame{
  border:4px solid rgba(255,255,255,0.06);
  padding:18px;
  border-radius:12px;
  background: var(--grid-bg);
  box-shadow: 0 12px 30px rgba(2,6,23,0.45);
}

.grid{
  display:grid;
  grid-template-columns: repeat(5, var(--cell-size));
  grid-template-rows: repeat(5, var(--cell-size));
  gap: var(--gap);
}

.cell{
  width:var(--cell-size);
  height:var(--cell-size);
  background: var(--cell-closed);
  border-radius:8px;
  display:flex;
  align-items:center;
  justify-content:center;
  font-weight:700;
  color: rgba(255,255,255,0.9);
  font-size:16px;
  user-select:none;
  border: 2px solid transparent;
  transform: scale(1);
  opacity:1;
  transition: transform 0.4s ease, opacity 0.5s ease, border 0.3s, background 0.4s;
}

.cell.open{
  border: 2px solid var(--cell-highlight); /* желтая рамка */
  background: var(--cell-open-bg); /* внутри чуть светлее */
  color: #FFD700; /* ⭐ */
  transform: scale(1.05);
}

.controls{
  display:flex;
  gap:12px;
  align-items:center;
  justify-content:center;
}

button.action{
  padding:10px 16px;
  border-radius:10px;
  border: none;
  font-weight:700;
  cursor:pointer;
  background: var(--accent);
  color: #022033;
  box-shadow: 0 6px 14px rgba(6,182,212,0.35);
  transition: transform .08s ease, box-shadow .2s;
}
button.action:hover{ transform: scale(1.05); box-shadow: 0 10px 20px rgba(6,182,212,0.45);}
button.action:active{ transform: scale(0.95); }
button.secondary{ background: #062e55; color: var(--text-light); }

button[disabled]{ opacity:.44; cursor:not-allowed; box-shadow:none; }

.status{ font-size:14px; opacity:.9; text-align:center; margin-top:6px; }

@media(max-width:520px){:root{--cell-size:46px;--gap:6px;}
.num-box{width: calc(var(--cell-size)*2.4); font-size:18px;}
.pm-btn{width:36px;height:36px;font-size:20px;}}
</style>
</head>
<body>
<div class="container">
<div class="number-control">
<button class="pm-btn" id="minusBtn">−</button>
<div class="num-box" id="numBox">1</div>
<button class="pm-btn" id="plusBtn">+</button>
</div>

<div class="frame">
<div class="grid" id="grid"></div>
</div>

<div class="controls">
<button id="newSignalBtn" class="secondary action">Новый сигнал</button>
<button id="recvSignalBtn" class="action" disabled>Получить сигнал</button>
</div>

<div class="status" id="status">Готово — нажмите "Получить сигнал" для открытия клеток.</div>
</div>

<script>
const ROWS=5,COLS=5,TOTAL_CELLS=ROWS*COLS;
const OPEN_COUNT = 4;
const DELAY_MS = 500;

const gridEl = document.getElementById('grid');
const newBtn = document.getElementById('newSignalBtn');
const recvBtn = document.getElementById('recvSignalBtn');
const statusEl = document.getElementById('status');
const numBox = document.getElementById('numBox');
const plusBtn = document.getElementById('plusBtn');
const minusBtn = document.getElementById('minusBtn');

let currentNum = 1;
plusBtn.addEventListener('click', () => { if(currentNum < 7){ currentNum += 2; numBox.textContent = currentNum; } });
minusBtn.addEventListener('click', () => { if(currentNum > 1){ currentNum -= 2; numBox.textContent = currentNum; } });

const cells = [];
for(let i=0; i<TOTAL_CELLS; i++){
  const d = document.createElement('div');
  d.className = 'cell';
  gridEl.appendChild(d);
  cells.push(d);
}

function pickRandomIndices(n){
  const arr = Array.from({length: TOTAL_CELLS}, (_, i) => i);
  for(let i=arr.length-1; i>0; i--){
    const j = Math.floor(Math.random()*(i+1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr.slice(0, n);
}

function resetGrid(){
  cells.forEach(c => { c.classList.remove('open'); c.textContent = ''; });
  statusEl.textContent = 'Готово — нажмите "Получить сигнал" для открытия клеток.';
}

newBtn.addEventListener('click', () => {
  resetGrid();
  recvBtn.disabled = false;
  newBtn.disabled = true;
  statusEl.textContent = 'Новый сигнал получен — нажмите "Получить сигнал".';
});

function sleep(ms){ return new Promise(r => setTimeout(r, ms)); }

recvBtn.addEventListener('click', async () => {
  recvBtn.disabled = true;
  newBtn.disabled = true;
  statusEl.textContent = 'Начало — открытие клеток...';
  
  const picks = pickRandomIndices(OPEN_COUNT); // только 4 клетки
  
  for(let i=0; i<picks.length; i++){
    const cell = cells[picks[i]];
    cell.classList.add('open');
    cell.textContent = '⭐';
    await sleep(DELAY_MS);
  }

  statusEl.textContent = `Окончание — открыто 4 клетки. Для нового раунда нажмите "Новый сигнал".`;
  newBtn.disabled = false;
  recvBtn.disabled = true;
});

resetGrid();
</script>
</body>
</html>

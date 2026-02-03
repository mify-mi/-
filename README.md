<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>人生ゲーム 拡張版</title>

<style>
  body { font-family: sans-serif; text-align: center; background: #eef; }
  #map { display: flex; flex-wrap: wrap; width: 420px; margin: 20px auto; }
  .cell {
    width: 60px;
    height: 60px;
    border: 2px solid #333;
    margin: 2px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: white;
    position: relative;
    font-size: 12px;
  }
  .p1 { background: #ffcccc; }
  .p2 { background: #ccddff; }
  .bad { background: #ffd6d6; }
</style>
</head>

<body>

<h1>🎲 人生ゲーム 拡張版</h1>

<p>現在のプレイヤー：<strong id="currentPlayer">1</strong></p>

<p>
👤1 💰<span id="money1">100</span>万 /
💼<span id="job1">未定</span> /
🏠<span id="house1">なし</span> /
🚗<span id="car1">なし</span>
</p>

<p>
👤2 💰<span id="money2">100</span>万 /
💼<span id="job2">未定</span> /
🏠<span id="house2">なし</span> /
🚗<span id="car2">なし</span>
</p>

<div id="map"></div>
<button onclick="rollDice()">サイコロを振る</button>
<div id="log"></div>

<script>
const goal = 20;
const jobs = [
  { name: "フリーター", salary: 10 },
  { name: "エンジニア", salary: 30 },
  { name: "医者", salary: 50 }
];

let players = [
  { pos: 0, money: 100, job: null, rank: 1, house: false, car: false },
  { pos: 0, money: 100, job: null, rank: 1, house: false, car: false }
];

let turn = 0;

// マップ生成
const map = document.getElementById("map");
for (let i = 0; i <= goal; i++) {
  const cell = document.createElement("div");
  cell.className = "cell";
  cell.id = "cell" + i;
  cell.textContent = i;
  if ([6, 14, 18].includes(i)) cell.classList.add("bad");
  map.appendChild(cell);
}

function rollDice() {
  const p = players[turn];
  if (p.pos >= goal) return;

  const dice = Math.floor(Math.random() * 6) + 1;
  p.pos += dice;
  if (p.pos > goal) p.pos = goal;

  let log = `🎲 ${dice}マス進んだ！`;

  // 職業決定
  if (p.pos === 3 && !p.job) {
    p.job = jobs[Math.floor(Math.random() * jobs.length)];
    log += `<br>💼 職業：${p.job.name}`;
  }

  // 車購入
  if (p.pos === 5 && !p.car && p.money >= 20) {
    p.money -= 20;
    p.car = true;
    log += "<br>🚗 車を購入！ -20万";
  }

  // 家購入
  if (p.pos === 10 && !p.house && p.money >= 50) {
    p.money -= 50;
    p.house = true;
    log += "<br>🏠 家を購入！ -50万";
  }

  // 職業ランクアップ
  if (p.pos === 15 && p.job) {
    p.rank++;
    log += "<br>📈 昇進！給料アップ";
  }

  // 給料
  if (p.pos % 5 === 0 && p.job) {
    let salary = p.job.salary * p.rank;
    if (p.car) salary += 10;
    p.money += salary;
    log += `<br>💰 給料 +${salary}万`;
  }

  // ハズレマス
  if ([6, 14, 18].includes(p.pos)) {
    const bad = Math.random();
    if (bad < 0.5) {
      p.money -= 20;
      log += "<br>💥 事故！ -20万";
    } else {
      p.job = null;
      p.rank = 1;
      log += "<br>😱 失業… 職業リセット";
    }
  }

  if (p.pos === goal) log += "<br><strong>🏁 ゴール！</strong>";

  updateUI();
  document.getElementById("log").innerHTML = log;

  turn = (turn + 1) % 2;
  document.getElementById("currentPlayer").textContent = turn + 1;
}

function updateUI() {
  for (let i = 0; i <= goal; i++) {
    document.getElementById("cell" + i).className = "cell";
    if ([6,14,18].includes(i)) document.getElementById("cell"+i).classList.add("bad");
  }

  players.forEach((p, i) => {
    document.getElementById("cell" + p.pos).classList.add(i === 0 ? "p1" : "p2");
  });

  ["1","2"].forEach((n,i)=>{
    document.getElementById("money"+n).textContent = players[i].money;
    document.getElementById("job"+n).textContent = players[i].job ? players[i].job.name+" Lv"+players[i].rank : "未定";
    document.getElementById("house"+n).textContent = players[i].house ? "あり" : "なし";
    document.getElementById("car"+n).textContent = players[i].car ? "あり" : "なし";
  });
}

updateUI();
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>人生ゲーム 200マス</title>

<style>
  body { font-family: sans-serif; background:#eef; text-align:center; }
  #map {
    display: grid;
    grid-template-columns: repeat(20, 1fr);
    gap: 2px;
    max-width: 800px;
    margin: 20px auto;
  }
  .cell {
    border: 1px solid #333;
    background: white;
    font-size: 10px;
    padding: 4px;
    height: 30px;
  }
  .p1 { background:#ffcccc; }
  .p2 { background:#ccddff; }
  .bad { background:#ffd6d6; }
</style>
</head>

<body>

<h1>🎲 人生ゲーム（200マス）</h1>

<p>現在プレイヤー：<b id="turn">1</b></p>
<p>🎲 サイコロ：<span id="dice">-</span></p>

<button onclick="startDice()">🎲 転がす</button>
<button onclick="stopDice()">✋ ストップ</button>

<p id="status"></p>

<p>
👤1 💰<span id="m1">100</span>万 |
💼<span id="j1">未定</span> |
🏠<span id="h1">なし</span> |
🚗<span id="c1">なし</span>
</p>

<p>
👤2 💰<span id="m2">100</span>万 |
💼<span id="j2">未定</span> |
🏠<span id="h2">なし</span> |
🚗<span id="c2">なし</span>
</p>

<div id="map"></div>

<script>
const GOAL = 200;
let diceInterval = null;
let diceValue = 1;

const jobs = [
  {name:"フリーター", base:10},
  {name:"エンジニア", base:30},
  {name:"医者", base:50}
];

const players = [
  {pos:0,money:100,job:null,rank:1,house:false,car:false},
  {pos:0,money:100,job:null,rank:1,house:false,car:false}
];

let turn = 0;

// マップ生成
const map = document.getElementById("map");
for(let i=1;i<=GOAL;i++){
  const c=document.createElement("div");
  c.className="cell";
  c.id="cell"+i;
  c.textContent=i;
  if(i%17===0) c.classList.add("bad");
  map.appendChild(c);
}

function startDice(){
  if(diceInterval) return;
  diceInterval = setInterval(()=>{
    diceValue = Math.floor(Math.random()*6)+1;
    document.getElementById("dice").textContent=diceValue;
  },80);
}

function stopDice(){
  if(!diceInterval) return;
  clearInterval(diceInterval);
  diceInterval=null;
  movePlayer(diceValue);
}

function movePlayer(d){
  const p = players[turn];
  p.pos += d;
  if(p.pos>GOAL) p.pos=GOAL;

  let log = `🎲 ${d}マス進んだ`;

  // 職業
  if(p.pos===10 && !p.job){
    p.job = jobs[Math.floor(Math.random()*jobs.length)];
    log+=` / 💼${p.job.name}`;
  }

  // 車
  if(p.pos===30 && !p.car && p.money>=30){
    p.money-=30; p.car=true;
    log+=" / 🚗車購入";
  }

  // 家
  if(p.pos===80 && !p.house && p.money>=80){
    p.money-=80; p.house=true;
    log+=" / 🏠家購入";
  }

  // 昇進
  if(p.pos%50===0 && p.job){
    p.rank++;
    log+=" / 📈昇進";
  }

  // 給料
  if(p.pos%20===0 && p.job){
    let pay = p.job.base*p.rank + (p.car?10:0);
    p.money+=pay;
    log+=` / 💰給料+${pay}`;
  }

  // ハズレマス
  if(p.pos%17===0){
    if(Math.random()<0.5){
      p.money-=30;
      log+=" / 💥事故-30";
    }else{
      p.job=null; p.rank=1;
      log+=" / 😱失業";
    }
  }

  document.getElementById("status").textContent=log;
  updateUI();

  if(p.pos===GOAL){
    document.getElementById("status").textContent+=" 🏁ゴール！";

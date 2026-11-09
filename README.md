# village-game
heme = null;
}
function updateMusicGain() {
  if (SETTINGS.soundOn && SETTINGS.musicVol > 0) {
    const theme = getMusicTheme();
    stopMusic();
    startMusic(theme);
  } else stopMusic();
}
function updateMusic() {
  const theme = getMusicTheme();
  if (currentMusicTheme !== theme) startMusic(theme);
}

// ==================== ЭТАПЫ ====================
const STAGES = [
  { name:'Заброшенная деревня', pop:0,    houses:0,   money:0,          food:0,      unlock:[] },
  { name:'Деревня',             pop:15,   houses:12,  money:150000,     food:200,    unlock:['farm','shop','well','lumberjack'] },
  { name:'Хутор',               pop:35,   houses:22,  money:500000,     food:600,    unlock:['mill','bakery','workshop','quarry'] },
  { name:'Посёлок',             pop:70,   houses:40,  money:1500000,    food:1500,   unlock:['school','clinic','tavern','mine'] },
  { name:'Маленький городок',   pop:140,  houses:70,  money:4000000,    food:4000,   unlock:['bank','factory','theater','smelter'] },
  { name:'Город',               pop:280,  houses:120, money:10000000,   food:10000,  unlock:['hospital','university','stadium','goldmine'] },
  { name:'Мегаполис',           pop:550,  houses:200, money:30000000,   food:25000,  unlock:['skyscraper','airport','mall'] },
  { name:'Столица',             pop:1100, houses:350, money:100000000,  food:60000,  unlock:[] }
];
const RES_DEFS = {
  wood:  { name:'🪵 Дерево',     color:'#8a5a2a' },
  stone: { name:'🪨 Камень',     color:'#8a8a9a' },
  iron:  { name:'⚙️ Железо',     color:'#7a8a9a' },
  gold:  { name:'✨ Золотая руда', color:'#d4a020' }
};
const BUILDING_DEFS = {
  farm:      { name:'🌾 Ферма',       cost:8000,    resCost:{wood:20},             prod:{food:3},    staff:1, buildTime:25, desc:'+3 еды/с' },
  shop:      { name:'🏪 Магазин',     cost:12000,   resCost:{wood:30},             prod:{money:8},   staff:1, buildTime:30, desc:'+8 денег/с' },
  well:      { name:'💧 Колодец',     cost:5000,    resCost:{stone:15},            prod:{food:1.5},  staff:0, buildTime:20, desc:'+1.5 еды/с' },
  lumberjack:{ name:'🪓 Лесопилка',   cost:10000,   resCost:{wood:10},             prod:{},          gather:{wood:1},   staff:1, buildTime:30, desc:'+1 дерева/с' },
  quarry:    { name:'⛏️ Каменоломня', cost:20000,   resCost:{wood:40},             prod:{},          gather:{stone:0.8},staff:2, buildTime:45, desc:'+0.8 камня/с' },
  mill:      { name:'⚙️ Мельница',    cost:25000,   resCost:{wood:50,stone:20},    prod:{food:8},    staff:2, buildTime:50, desc:'+8 еды/с' },
  bakery:    { name:'🥖 Пекарня',     cost:35000,   resCost:{wood:40,stone:30},    prod:{money:20},  staff:2, buildTime:45, desc:'+20 денег/с' },
  workshop:  { name:'🔧 Мастерская',  cost:45000,   resCost:{wood:60,stone:40},    prod:{money:25},  staff:2, buildTime:55, desc:'+25 денег/с' },
  mine:      { name:'⛏️ Шахта',       cost:120000,  resCost:{wood:100,stone:80},   prod:{},          gather:{iron:0.5}, staff:3, buildTime:70, desc:'+0.5 железа/с' },
  school:    { name:'🏫 Школа',       cost:100000,  resCost:{wood:120,stone:100},  prod:{},          staff:3, buildTime:60, bonus:'money', bonusVal:0.06, desc:'+6% к деньгам/шт.' },
  clinic:    { name:'🏥 Поликлиника', cost:130000,  resCost:{wood:100,stone:140},  prod:{},          staff:3, buildTime:65, bonus:'food',  bonusVal:0.06, desc:'+6% к еде/шт.' },
  tavern:    { name:'🍺 Таверна',     cost:90000,   resCost:{wood:80,stone:60},    prod:{money:40},  staff:2, buildTime:50, desc:'+40 денег/с' },
  smelter:   { name:'🔥 Плавильня',   cost:300000,  resCost:{wood:150,stone:200,iron:50}, prod:{}, gather:{gold:0.3}, staff:5, buildTime:80, desc:'+0.3 золотой руды/с' },
  bank:      { name:'🏦 Банк',        cost:350000,  resCost:{wood:200,stone:250,iron:80}, prod:{money:130}, staff:4, buildTime:75, desc:'+130 денег/с' },
  factory:   { name:'🏭 Завод',       cost:500000,  resCost:{wood:250,stone:300,iron:120}, prod:{money:200}, staff:8, buildTime:90, desc:'+200 денег/с' },
  theater:   { name:'


🎭 Театр',       cost:450000,  resCost:{wood:200,stone:200,iron:100}, prod:{money:100}, staff:4, buildTime:75, desc:'+100 денег/с' },
  goldmine:  { name:'✨ Золотой рудник', cost:900000, resCost:{wood:300,stone:300,iron:150}, prod:{}, gather:{gold:0.8}, staff:6, buildTime:100, desc:'+0.8 золотой руды/с' },
  hospital:  { name:'🚑 Госпиталь',   cost:1500000, resCost:{wood:500,stone:600,iron:300}, prod:{}, staff:10, buildTime:100, bonus:'food', bonusVal:0.10, desc:'+10% к еде/шт.' },
  university:{ name:'🎓 Университет', cost:2200000, resCost:{wood:600,stone:700,iron:400}, prod:{}, staff:12, buildTime:110, bonus:'money', bonusVal:0.10, desc:'+10% к деньгам/шт.' },
  stadium:   { name:'🏟️ Стадион',     cost:2800000, resCost:{wood:700,stone:800,iron:500}, prod:{money:600}, staff:15, buildTime:105, desc:'+600 денег/с' },
  skyscraper:{ name:'🏢 Небоскрёб',   cost:6000000, resCost:{wood:1000,stone:1500,iron:800,gold:100}, prod:{money:1400}, staff:20, buildTime:115, desc:'+1400 денег/с' },
  mall:      { name:'🛒 ТРЦ',         cost:8000000, resCost:{wood:1200,stone:1800,iron:1000,gold:150}, prod:{money:2000}, staff:20, buildTime:110, desc:'+2000 денег/с' },
  airport:   { name:'✈️ Аэропорт',    cost:10000000,resCost:{wood:1500,stone:2000,iron:1200,gold:250}, prod:{money:2600}, staff:25, buildTime:120, desc:'+2600 денег/с' }
};
const SEASONS = [
  { name:'🌱 Весна',  color:'#3a6a3a', mult:{food:1.2, money:1.0, gather:1.0},  desc:'+20% к еде' },
  { name:'☀️ Лето',   color:'#4a8a3a', mult:{food:1.5, money:1.1, gather:1.0},  desc:'+50% к еде' },
  { name:'🍂 Осень',  color:'#6a5a2a', mult:{food:1.0, money:1.2, gather:1.1},  desc:'+20% к деньгам' },
  { name:'❄️ Зима',   color:'#4a5a6a', mult:{food:0.5, money:0.9, gather:0.7},  desc:'−50% к еде' }
];

// Дом строителей — новая цена 70 000
const BUILDER_HOUSE_COST = { money: 70000, wood: 100, stone: 80 };
const BUILDER_HOUSE_SLOTS = 4;
const BUILDER_HOUSE_BUILD_TIME = 60;

// ==================== СОСТОЯНИЕ ====================
function newGameState() {
  return {
    money:1000000, food:300, pop:10, popCap:10,
    houses:10, abandoned:20,
    resources:{wood:50, stone:30, iron:0, gold:0},
    buildings:{farm:1, shop:1},
    builderHouses: 1,
    stage:0, loan:1000000, taxRate:10, unrest:0,
    timeMin:360, day:1, season:0, seasonProgress:0,
    construction: [],
    gameTime: 0,
    lastTick:Date.now()
  };
}
let S = newGameState();
let gameRunning = false;

// ==================== УТИЛИТЫ ====================
function fmt(n) {
  if (n>=1e9) return (n/1e9).toFixed(2)+'B';
  if (n>=1e6) return (n/1e6).toFixed(2)+'M';
  if (n>=1e3) return (n/1e3).toFixed(1)+'K';
  return Math.floor(n).toString();
}
function fmtTime(sec) {
  if (sec < 60) return Math.ceil(sec) + 'с';
  return Math.floor(sec/60) + 'м ' + Math.ceil(sec%60) + 'с';
}
function addLog(msg, cls='') {
  const log = document.getElementById('log');
  if (!log) return;
  const d = document.createElement('div');
  if (cls) d.className = cls;
  const t = new Date().toLocaleTimeString('ru-RU',{hour:'2-digit',minute:'2-digit',second:'2-digit'});
  d.textContent = `[${t}] ${msg}`;
  log.prepend(d);
  while (log.children.length>60) log.removeChild(log.lastChild);
}
function toast(msg) {
  const el = document.createElement('div');
  el.className = 'toast';
  el.textContent = msg;
  document.body.appendChild(el);
  setTimeout(() => el.remove(), 3500);
}
function totalStaffNeeded() {
  let n=0;
  for (const [id,cnt] of Object.entries(S.buildings)) n += (BUILDING_DEFS[id]?.staff||0)*cnt;
  return n;
}
function totalStaffInConstruction() {
  let n=0;
  for (const c of S.construction) {
    if (c.id === 'house_restore') continue;
    if (c.id === 'builder_house') continue;
    n += (BUILDING_DEFS[c.id]?.staff||0);
  }
  return n;
}
function getBuildSlots() {
  return S.builderHouses * BUILDER_HOUSE_SLOTS;
}
function getFreeBuildSlots() {
  return getBuildSlots() - S.construction.length;
}
function canAffordResources(resCost) {
  for (const [res,a


mt] of Object.entries(resCost||{})) if ((S.resources[res]||0)<amt) return false;
  return true;
}
function payResources(resCost) { for (const [res,amt] of Object.entries(resCost||{})) S.resources[res]-=amt; }
function getBonuses() {
  const b={money:1,food:1};
  for (const [id,cnt] of Object.entries(S.buildings)) {
    const def=BUILDING_DEFS[id];
    if (def && def.bonus) b[def.bonus]+=def.bonusVal*cnt;
  }
  return b;
}
function getSeason() { return SEASONS[S.season]; }
function getDayPhase() {
  const t=S.timeMin/1440;
  if (t>=6/24 && t<18/24) return 'day';
  if (t>=18/24 && t<22/24) return 'evening';
  return 'night';
}
function getProduction() {
  const bonuses=getBonuses(), season=getSeason(), phase=getDayPhase();
  const nightMult = phase==='night'?0.5:1;
  const p={money:0,food:0};
  for (const [id,cnt] of Object.entries(S.buildings)) {
    const def=BUILDING_DEFS[id];
    if (!def||!def.prod) continue;
    if (def.prod.money) p.money+=def.prod.money*cnt*bonuses.money*season.mult.money*nightMult;
    if (def.prod.food)  p.food +=def.prod.food *cnt*bonuses.food *season.mult.food;
  }
  return p;
}
function getGathering() {
  const season=getSeason();
  const g={wood:0,stone:0,iron:0,gold:0};
  for (const [id,cnt] of Object.entries(S.buildings)) {
    const def=BUILDING_DEFS[id];
    if (!def||!def.gather) continue;
    for (const [res,amt] of Object.entries(def.gather)) g[res]+=amt*cnt*season.mult.gather;
  }
  return g;
}
function getFoodConsumption() { return S.pop*0.12; }
function getTaxIncome() {
  const base=S.pop*0.5 + Object.values(S.buildings).reduce((a,b)=>a+b,0)*2;
  return base*(S.taxRate/100)*Math.max(0,1-S.unrest/150);
}
function getStageProgress() {
  const next=STAGES[S.stage+1];
  if (!next) return 1;
  return Math.min(S.pop/next.pop, S.houses/next.houses, S.money/next.money, S.food/next.food, 1);
}
function checkStageUp() {
  const next=STAGES[S.stage+1];
  if (!next) return;
  if (S.pop>=next.pop && S.houses>=next.houses && S.money>=next.money && S.food>=next.food) {
    S.stage++;
    const st=STAGES[S.stage];
    addLog(`🎉 Новый этап: ${st.name}!`,'good');
    playLevelUp();
    if (st.unlock.length) addLog(`Открыто: ${st.unlock.map(id=>BUILDING_DEFS[id].name).join(', ')}`,'good');
    showModal(`Этап: ${st.name}`, st.unlock.length
      ? `Открыты постройки: ${st.unlock.map(id=>BUILDING_DEFS[id].name).join(', ')}.`
      : 'Ты достиг Столицы!');
    updateMusic();
  }
}
function showModal(title,text) {
  document.getElementById('modalTitle').textContent=title;
  document.getElementById('modalText').textContent=text;
  document.getElementById('modal').classList.add('show');
}
function isUnlocked(id) {
  if (id==='farm'||id==='shop') return true;
  for (let i=0;i<=S.stage;i++) if (STAGES[i].unlock.includes(id)) return true;
  return false;
}
function setTax(v) {
  S.taxRate=+v;
  document.getElementById('taxVal').textContent=v+'%';
}

// ==================== СТРОИТЕЛЬСТВО ====================
function getSpeedUpCost(c) {
  // Стоимость ускорения: 20 денег за секунду (10 сек = 200)
  return Math.ceil(c.remaining * 20);
}
function speedUpConstruction(index) {
  if (!gameRunning) return;
  const c = S.construction[index];
  if (!c) return;
  const cost = getSpeedUpCost(c);
  if (S.money < cost) { addLog('Не хватает денег на ускорение','bad'); playClick(); return; }
  if (!confirm(`Ускорить строительство?\nСтоимость: ${fmt(cost)} денег\nОсталось: ${fmtTime(c.remaining)}`)) return;
  S.money -= cost;
  c.remaining = 0;
  playSpeedUp();
  addLog(`⚡ Ускорено: ${c.id === 'house_restore' ? 'восстановление дома' : c.id === 'builder_house' ? 'дом строителей' : BUILDING_DEFS[c.id].name}`, 'good');
  render();
  save();
}

// ==================== ДЕЙСТВИЯ ====================
function build(id) {
  if (!gameRunning) return;
  const def=BUILDING_DEFS[id];
  if (!def) return;
  if (getFreeBuildSlots() <= 0) { addLog(`Нет свободных слотов строительства (нужен дом строителей)`, 'bad'); playClick(); return; }
  if (S


.money<def.cost) { addLog('Не хватает денег','bad'); playClick(); return; }
  if (!canAffordResources(def.resCost)) { addLog(`Не хватает ресурсов для ${def.name}`,'bad'); playClick(); return; }
  const owned=S.buildings[id]||0;
  if (owned===0 && !isUnlocked(id)) return;
  const free=S.pop-totalStaffNeeded()-totalStaffInConstruction();
  if (def.staff>0 && free<def.staff) { addLog(`Нет рабочих для ${def.name}`,'bad'); playClick(); return; }
  const resStr = def.resCost ? Object.entries(def.resCost).map(([r,a])=>RES_DEFS[r].name.split(' ')[0]+a).join(' ') : '';
  if (SETTINGS.confirm && !confirm(`Начать строительство: ${def.name}?\n💰${fmt(def.cost)} ${resStr}\n⏱️ ${fmtTime(def.buildTime)}\nСлотов: ${getFreeBuildSlots()}/${getBuildSlots()}`)) return;

  S.money-=def.cost;
  payResources(def.resCost);
  S.construction.push({ id, remaining: def.buildTime, total: def.buildTime });
  addLog(`🏗️ Начато строительство: ${def.name} (${fmtTime(def.buildTime)})`, 'build');
  playBuild();
  updateMusic();
  render();
  save();
}
function buildBuilderHouse() {
  if (!gameRunning) return;
  if (getFreeBuildSlots() <= 0) { addLog(`Нет свободных слотов строительства`, 'bad'); playClick(); return; }
  const c = BUILDER_HOUSE_COST;
  if (S.money < c.money) { addLog('Не хватает денег на дом строителей','bad'); playClick(); return; }
  if (!canAffordResources({wood:c.wood, stone:c.stone})) { addLog('Не хватает дерева/камня','bad'); playClick(); return; }
  if (SETTINGS.confirm && !confirm(`Построить дом строителей?\n💰${fmt(c.money)} 🪵${c.wood} 🪨${c.stone}\n⏱️ ${fmtTime(BUILDER_HOUSE_BUILD_TIME)}\nДаст +${BUILDER_HOUSE_SLOTS} слота строительства`)) return;
  S.money -= c.money;
  payResources({wood:c.wood, stone:c.stone});
  S.construction.push({ id: 'builder_house', remaining: BUILDER_HOUSE_BUILD_TIME, total: BUILDER_HOUSE_BUILD_TIME });
  addLog(`🏗️ Начато строительство дома строителей (${fmtTime(BUILDER_HOUSE_BUILD_TIME)})`, 'build');
  playBuild();
  updateMusic();
  render();
  save();
}
function restoreHouse() {
  if (!gameRunning) return;
  if (getFreeBuildSlots() <= 0) { addLog(`Нет свободных слотов строительства`, 'bad'); playClick(); return; }
  if (S.abandoned<=0) { addLog('Нет заброшенных домов','warn'); return; }
  if (S.money<50000) { addLog('Не хватает денег','bad'); playClick(); return; }
  if (!canAffordResources({wood:30,stone:20})) { addLog('Не хватает дерева/камня','bad'); playClick(); return; }
  if (SETTINGS.confirm && !confirm('Восстановить дом?\n💰50000 🪵30 🪨20\n⏱️ 40с\nСлотов: '+getFreeBuildSlots()+'/'+getBuildSlots())) return;
  S.money-=50000; payResources({wood:30,stone:20});
  S.construction.push({ id: 'house_restore', remaining: 40, total: 40 });
  addLog('🏗️ Начато восстановление дома (40с)', 'build');
  playBuild();
  updateMusic();
  render();
  save();
}
function attractPeople() {
  if (!gameRunning) return;
  if (S.money<20000) return;
  if (S.popCap<=S.pop) { addLog('Нет свободных мест','warn'); return; }
  S.money-=20000; S.pop++;
  addLog('Привлечён житель','good'); playClick(); render(); save();
}
function payLoan() {
  if (!gameRunning) return;
  if (S.loan<=0) return;
  const amt=Math.min(100000,S.loan,Math.floor(S.money));
  if (amt<=0) return;
  S.money-=amt; S.loan-=amt;
  addLog(`Выплачено по кредиту: ${fmt(amt)}`,'good'); playClick(); render(); save();
}
function sellResource(res,amt) {
  if (!gameRunning) return;
  const prices={wood:50,stone:80,iron:200,gold:600};
  const have=Math.floor(S.resources[res]||0);
  const sell=Math.min(amt,have);
  if (sell<=0) return;
  S.resources[res]-=sell;
  S.money+=sell*prices[res];
  addLog(`Продано ${RES_DEFS[res].name} ×${sell} → +${fmt(sell*prices[res])}`,'good');
  playClick(); render(); save();
}

// ==================== КАРТА ====================
const canvas=document.getElementById('mapCanvas');
const ctx=canvas.getContext('2d');
const TILE=40;
let particles=[];

function qualityScale() {
  return SETTINGS.quality==='low' ? 0.7 : SETTINGS.quality===


'high' ? 1 : 0.85;
}
function resizeCanvas() {
  if (!canvas) return;
  const q = qualityScale();
  canvas.width = Math.floor(620*q);
  canvas.height = Math.floor(400*q);
  if (gameRunning) drawMap();
}

function drawHouse(x,y,s,abandoned) {
  ctx.fillStyle=abandoned?'#4a4a4a':'#c8a878';
  ctx.fillRect(x+s*0.15,y+s*0.4,s*0.7,s*0.5);
  ctx.fillStyle=abandoned?'#2a2a2a':'#a0522d';
  ctx.beginPath(); ctx.moveTo(x+s*0.5,y+s*0.1); ctx.lineTo(x+s*0.1,y+s*0.4); ctx.lineTo(x+s*0.9,y+s*0.4); ctx.closePath(); ctx.fill();
  ctx.fillStyle=abandoned?'#222':(getDayPhase()==='night'?'#ffcc44':'#ffe97f');
  ctx.fillRect(x+s*0.35,y+s*0.55,s*0.12,s*0.12);
  ctx.fillRect(x+s*0.55,y+s*0.55,s*0.12,s*0.12);
  ctx.fillStyle=abandoned?'#1a1a1a':'#5a3a1a';
  ctx.fillRect(x+s*0.42,y+s*0.7,s*0.16,s*0.2);
}
function drawBuilderHouse(x,y,s) {
  ctx.fillStyle='#b0a080';
  ctx.fillRect(x+s*0.15,y+s*0.4,s*0.7,s*0.5);
  ctx.fillStyle='#d04040';
  ctx.beginPath(); ctx.moveTo(x+s*0.5,y+s*0.1); ctx.lineTo(x+s*0.1,y+s*0.4); ctx.lineTo(x+s*0.9,y+s*0.4); ctx.closePath(); ctx.fill();
  ctx.fillStyle='#5a3a1a';
  ctx.fillRect(x+s*0.35,y+s*0.55,s*0.1,s*0.08);
  ctx.fillStyle='#888';
  ctx.fillRect(x+s*0.42,y+s*0.5,s*0.08,s*0.12);
  ctx.fillStyle=getDayPhase()==='night'?'#ffcc44':'#ffe97f';
  ctx.fillRect(x+s*0.6,y+s*0.55,s*0.12,s*0.12);
}
function drawConstruction(x,y,s,progress) {
  ctx.fillStyle='#6a5a3a';
  ctx.fillRect(x+s*0.15,y+s*0.35,s*0.7,s*0.55);
  ctx.strokeStyle='#d0b060'; ctx.lineWidth=2;
  ctx.beginPath();
  ctx.moveTo(x+s*0.2,y+s*0.9); ctx.lineTo(x+s*0.2,y+s*0.4);
  ctx.lineTo(x+s*0.8,y+s*0.4); ctx.lineTo(x+s*0.8,y+s*0.9);
  ctx.moveTo(x+s*0.35,y+s*0.4); ctx.lineTo(x+s*0.35,y+s*0.9);
  ctx.moveTo(x+s*0.65,y+s*0.4); ctx.lineTo(x+s*0.65,y+s*0.9);
  ctx.stroke();
  for (let i=0;i<4;i++) {
    ctx.fillStyle = i%2 ? '#fff' : '#d04040';
    ctx.fillRect(x+s*0.15+i*s*0.18, y+s*0.28, s*0.18, s*0.06);
  }
  if (progress !== undefined) {
    ctx.fillStyle='rgba(0,0,0,0.5)';
    ctx.fillRect(x+s*0.1, y+s*0.92, s*0.8, s*0.06);
    ctx.fillStyle='#7fff9f';
    ctx.fillRect(x+s*0.1, y+s*0.92, s*0.8*progress, s*0.06);
  }
}
function drawFarm(x,y,s) {
  const season=S.season;
  const fc=season===3?'#d8e8f0':(season===2?'#a08040':'#7ab04a');
  ctx.fillStyle=fc; ctx.fillRect(x+s*0.1,y+s*0.3,s*0.8,s*0.6);
  if (season!==3) {
    ctx.fillStyle=season===2?'#806020':'#5a8a2a';
    for (let i=0;i<3;i++) ctx.fillRect(x+s*0.15,y+s*0.4+i*s*0.15,s*0.7,s*0.08);
  }
  ctx.fillStyle='#d4a060'; ctx.fillRect(x+s*0.3,y+s*0.15,s*0.4,s*0.3);
  ctx.fillStyle='#a0522d';
  ctx.beginPath(); ctx.moveTo(x+s*0.5,y+s*0.05); ctx.lineTo(x+s*0.25,y+s*0.15); ctx.lineTo(x+s*0.75,y+s*0.15); ctx.closePath(); ctx.fill();
}
function drawShop(x,y,s) {
  ctx.fillStyle='#b0b0c0'; ctx.fillRect(x+s*0.15,y+s*0.35,s*0.7,s*0.55);
  for (let i=0;i<5;i++){ctx.fillStyle=i%2?'#fff':'#d04040';ctx.fillRect(x+s*0.15+i*s*0.14,y+s*0.25,s*0.14,s*0.1);}
  ctx.fillStyle=getDayPhase()==='night'?'#ffcc44':'#7fc4ff';
  ctx.fillRect(x+s*0.25,y+s*0.5,s*0.5,s*0.2);
}
function drawTree(x,y,s) {
  const season=S.season;
  ctx.fillStyle='#5a3a1a'; ctx.fillRect(x+s*0.45,y+s*0.55,s*0.1,s*0.35);
  if (season===3) {
    ctx.fillStyle='#e8f0f5'; ctx.beginPath(); ctx.arc(x+s*0.5,y+s*0.4,s*0.28,0,Math.PI*2); ctx.fill();
  } else {
    const leaf=season===2?'#c08020':(season===0?'#4aaa4a':'#2a7a2a');
    ctx.fillStyle=leaf; ctx.beginPath(); ctx.arc(x+s*0.5,y+s*0.4,s*0.28,0,Math.PI*2); ctx.fill();
    ctx.fillStyle=season===2?'#d0a030':'#3a9a3a';
    ctx.beginPath(); ctx.arc(x+s*0.45,y+s*0.32,s*0.18,0,Math.PI*2); ctx.fill();
  }
}
function drawWell(x,y,s) {
  ctx.fillStyle='#7a7a8a'; ctx.fillRect(x+s*0.3,y+s*0.5,s*0.4,s*0.35);
  ctx.fillStyle='#5a3a1a'; ctx.fillRect(x+s*0.25,y+s*0.35,s*0.5,s*0.1);
  ctx.fillStyle='#4a90d0'; ctx.beginPath(); ctx.arc(x+s*0.5,y+s*0.55,s*0.12,0,Math.PI*2); ctx.fill();
}
function drawBuilding(x,y,s,type) {
  const colors={lumberjack:'#8a6a3a',quarry:'#8a8a9a',mill:'#8a6a3a',bakery:'#d4a060',workshop:'#8a8a9a',mine


:'#5a5a6a',smelter:'#a04020',school:'#d0b040',clinic:'#e0e0f0',tavern:'#a0603a',bank:'#e0d080',factory:'#7a7a8a',theater:'#a04060',goldmine:'#c09030',hospital:'#e0e0f0',university:'#d0b040',stadium:'#40a040',skyscraper:'#8090c0',airport:'#a0a0b0',mall:'#c08040'};
  ctx.fillStyle=colors[type]||'#888';
  ctx.fillRect(x+s*0.15,y+s*0.25,s*0.7,s*0.65);
  ctx.fillStyle='rgba(0,0,0,0.3)'; ctx.fillRect(x+s*0.15,y+s*0.25,s*0.7,s*0.1);
  ctx.fillStyle=getDayPhase()==='night'?'#ffcc44':'#ffe97f';
  for (let i=0;i<2;i++) for (let j=0;j<2;j++) ctx.fillRect(x+s*0.25+i*s*0.25,y+s*0.45+j*s*0.22,s*0.15,s*0.12);
}

function drawMap() {
  if (!gameRunning) return;
  const W=canvas.width, H=canvas.height;
  const season=getSeason(), phase=getDayPhase();
  ctx.fillStyle=season.color; ctx.fillRect(0,0,W,H);
  if (SETTINGS.shadows) {
    if (phase==='night') { ctx.fillStyle='rgba(10,20,40,0.5)'; ctx.fillRect(0,0,W,H); }
    else if (phase==='evening') { ctx.fillStyle='rgba(80,30,10,0.2)'; ctx.fillRect(0,0,W,H); }
  }
  if (SETTINGS.grid) {
    for (let x=0;x<W;x+=TILE) for (let y=0;y<H;y+=TILE) {
      ctx.fillStyle=(x/TILE+y/TILE)%2?'rgba(0,0,0,0.05)':'rgba(255,255,255,0.03)';
      ctx.fillRect(x,y,TILE,TILE);
    }
  }
  const slots=[];
  for (let y=1;y<H/TILE-1;y++) for (let x=1;x<W/TILE-1;x++) slots.push({x:x*TILE,y:y*TILE});
  let idx=0;
  for (let i=0;i<S.builderHouses && idx<slots.length;i++) {
    const sl=slots[idx++];
    drawBuilderHouse(sl.x+2, sl.y+2, TILE-4);
  }
  const totalHouses=S.houses+S.abandoned;
  const houseSlots=slots.slice(idx, idx+Math.min(totalHouses,70));
  for (let i=0;i<totalHouses && idx<slots.length;i++,idx++) {
    const sl=slots[idx]; drawHouse(sl.x+2,sl.y+2,TILE-4,i>=S.houses);
  }
  for (const [id,cnt] of Object.entries(S.buildings)) {
    for (let i=0;i<Math.min(cnt,6);i++) {
      if (idx>=slots.length) break;
      const sl=slots[idx++];
      if (id==='farm') drawFarm(sl.x+2,sl.y+2,TILE-4);
      else if (id==='shop') drawShop(sl.x+2,sl.y+2,TILE-4);
      else if (id==='well') drawWell(sl.x+2,sl.y+2,TILE-4);
      else drawBuilding(sl.x+2,sl.y+2,TILE-4,id);
    }
  }
  for (const c of S.construction) {
    if (idx>=slots.length) break;
    const sl=slots[idx++];
    const pct = (1 - c.remaining/c.total);
    drawConstruction(sl.x+2, sl.y+2, TILE-4, pct);
  }
  const treeCount = SETTINGS.quality==='low'?5:SETTINGS.quality==='high'?30:20;
  for (let i=0;i<treeCount;i++) {
    const x=(i*97)%W, y=(i*53)%(H-TILE);
    if (y<TILE*0.8||x<TILE*0.8) drawTree(x,y,TILE-4);
  }
  if (SETTINGS.particles) {
    particles.forEach(p => {
      ctx.globalAlpha = p.life/p.max;
      ctx.fillStyle = p.color;
      ctx.fillRect(p.x, p.y, 3, 3);
    });
    ctx.globalAlpha = 1;
  }
}

// ==================== ВРЕМЯ / СЕЗОН ====================
function updateTime(dt) {
  S.timeMin += dt*60*SETTINGS.scrollSpeed;
  if (S.timeMin>=1440) {
    S.timeMin-=1440; S.day++;
    S.seasonProgress+=1/7;
    if (S.seasonProgress>=1) {
      S.seasonProgress=0;
      S.season=(S.season+1)%4;
      const s=getSeason();
      addLog(`🍃 Смена сезона: ${s.name} — ${s.desc}`,'warn');
      playSeasonChange();
    }
  }
}
function formatTime() {
  const h=Math.floor(S.timeMin/60), m=Math.floor(S.timeMin%60);
  return String(h).padStart(2,'0')+':'+String(m).padStart(2,'0');
}
function updateClockUI() {
  if (!gameRunning) return;
  document.getElementById('clockTime').textContent=formatTime();
  document.getElementById('clockSeason').textContent=`${getSeason().name} · День ${S.day}`;
}

// ==================== ИГРОВОЙ ЦИКЛ ====================
let lastRender=0, lastMapDraw=0;
function tick() {
  if (!gameRunning) return;
  const now=Date.now();
  const dt=Math.min((now-S.lastTick)/1000,2);
  S.lastTick=now;
  S.gameTime += dt;
  updateTime(dt);
  const prod=getProduction();
  S.money+=prod.money*dt;
  S.money+=getTaxIncome()*dt;
  const foodNet=prod.food-getFoodConsumption();
  S.food+=foodNet*dt;
  const g=getGathering();
  for (const [res,amt] of


Object.entries(g)) S.resources[res]+=amt*dt;
  const targetUnrest=S.taxRate*3-(S.food>0?10:-10);
  S.unrest+=(targetUnrest-S.unrest)*0.02*dt*10;
  S.unrest=Math.max(0,Math.min(100,S.unrest));
  if (S.loan>0) S.loan*=(1+0.00015*dt);

  if (S.construction.length > 0) {
    S.construction.forEach(c => { c.remaining -= dt; });
    for (let i = S.construction.length - 1; i >= 0; i--) {
      const c = S.construction[i];
      if (c.remaining <= 0) {
        if (c.id === 'house_restore') {
          S.abandoned--; S.houses++; S.popCap++;
          addLog('✅ Дом восстановлен (+1 место)', 'good');
          toast('🏠 Дом восстановлен!');
        } else if (c.id === 'builder_house') {
          S.builderHouses++;
          addLog(`✅ Дом строителей построен! +${BUILDER_HOUSE_SLOTS} слотов (всего: ${getBuildSlots()})`, 'good');
          toast(`🏗️ Дом строителей готов! +${BUILDER_HOUSE_SLOTS} слота`);
        } else {
          S.buildings[c.id] = (S.buildings[c.id]||0) + 1;
          addLog(`✅ Построено: ${BUILDING_DEFS[c.id].name}`, 'good');
          toast(`✅ ${BUILDING_DEFS[c.id].name} готово!`);
        }
        playBuildComplete();
        S.construction.splice(i, 1);
      }
    }
    updateMusic();
  }

  if (S.food<0) {
    S.food=0;
    if (Math.random()<0.01*dt*10 && S.pop>1) { S.pop--; addLog('Житель ушёл из-за голода','bad'); }
  }
  if (S.unrest>80 && Math.random()<0.005*dt*10 && S.pop>1) {
    S.pop--; addLog('Житель ушёл из-за налогов','bad');
  }
  if (Math.random()<0.008*dt*10) {
    const evs=[
      ()=>{const m=8000+S.stage*5000;S.money+=m;addLog(`Ярмарка! +${fmt(m)}`,'good');},
      ()=>{S.resources.wood+=30;addLog('Найден склад дерева (+30)','good');},
      ()=>{S.resources.iron+=15;addLog('Найдена жила железа (+15)','good');},
      ()=>{const m=Math.min(15000,S.money);S.money-=m;addLog(`Штраф: -${fmt(m)}`,'bad');},
      ()=>{if(S.popCap>S.pop){S.pop++;addLog('Перехожий остался (+1)','good');}}
    ];
    evs[Math.floor(Math.random()*evs.length)]();
  }
  checkStageUp();
  if (now-lastRender>200) { render(); updateClockUI(); lastRender=now; }
  if (now-lastMapDraw>SETTINGS.mapFreq) {
    if (SETTINGS.particles) {
      particles.forEach(p=>{p.x+=p.vx;p.y+=p.vy;p.life--;});
      particles=particles.filter(p=>p.life>0);
      if (Math.random()<0.5) {
        const x=Math.random()*canvas.width, y=Math.random()*canvas.height;
        particles.push({x,y,vx:(Math.random()-0.5)*0.5,vy:-0.5-Math.random()*0.5,life:30,max:30,color:getDayPhase()==='night'?'#ffcc44':'#ffe97f'});
        if (particles.length>100) particles.shift();
      }
    } else particles=[];
    drawMap();
    lastMapDraw=now;
  }
  save();
}

// ==================== ОТРИСОВКА UI ====================
function render() {
  if (!gameRunning) return;
  const st=STAGES[S.stage];
  document.getElementById('stageName').textContent=st.name;
  document.getElementById('stageBar').style.width=(getStageProgress()*100).toFixed(1)+'%';
  const next=STAGES[S.stage+1];
  document.getElementById('stageHint').textContent=next
    ?`До «${next.name}»: население ${S.pop}/${next.pop}, дома ${S.houses}/${next.houses}, деньги ${fmt(S.money)}/${fmt(next.money)}, еда ${fmt(S.food)}/${fmt(next.food)}`
    :'Максимальный этап!';
  const prod=getProduction();
  const foodNet=prod.food-getFoodConsumption();
  const taxInc=getTaxIncome();
  const g=getGathering();
  const freeSlots = getFreeBuildSlots();
  const statEls=[
    `<div class="stat"><div class="label">💰 Деньги</div><div class="value">${fmt(S.money)}</div><div class="delta">+${(prod.money+taxInc).toFixed(1)}/с</div></div>`,
    `<div class="stat"><div class="label">🍎 Еда</div><div class="value">${fmt(S.food)}</div><div class="delta" style="color:${foodNet>=0?'#7fff9f':'#ff7f7f'}">${foodNet>=0?'+':''}${foodNet.toFixed(1)}/с</div></div>`,
    `<div class="stat"><div class="label">👥 Население</div><div class="value">${S.pop}/${S.popCap}</div><div class="delta">раб.: ${totalStaffNeeded()}</div></div>`,
    `<div cl


ass="stat"><div class="label">🏠 Дома</div><div class="value">${S.houses}</div><div class="delta">забр.: ${S.abandoned}</div></div>`,
    `<div class="stat builder"><div class="label">🏗️ Дома строителей</div><div class="value">${S.builderHouses}</div><div class="delta">слотов: ${freeSlots}/${getBuildSlots()}</div></div>`,
    `<div class="stat ${S.loan>0?'negative':''}"><div class="label">💳 Кредит</div><div class="value">${fmt(S.loan)}</div></div>`
  ];
  for (const [res,def] of Object.entries(RES_DEFS)) {
    const val=Math.floor(S.resources[res]||0);
    const rate=g[res]||0;
    statEls.push(`<div class="stat"><div class="label">${def.name}</div><div class="value">${fmt(val)}</div><div class="delta">${rate>0?'+'+rate.toFixed(1)+'/с':'—'}</div></div>`);
  }
  document.getElementById('stats').innerHTML=statEls.join('');
  document.getElementById('taxIncome').textContent=taxInc.toFixed(1);
  document.getElementById('taxUnrest').textContent=S.unrest.toFixed(0)+'%';
  document.getElementById('unrestBar').style.width=S.unrest+'%';

  const slotsEl = document.getElementById('buildSlots');
  const usedSlots = S.construction.length;
  slotsEl.innerHTML = `
    <span>🏗️ Слоты строительства: <b>${usedSlots}/${getBuildSlots()}</b></span>
    <span class="${freeSlots>0?'free':'full'}">${freeSlots>0?`свободно ${freeSlots}`:'все заняты'}</span>
  `;

  const bEl=document.getElementById('buildings');
  bEl.innerHTML='';
  S.construction.forEach((c, i) => {
    const def = c.id==='house_restore' ? {name:'🏚️ Восстановление дома', staff:0} 
              : c.id==='builder_house' ? {name:'🏗️ Дом строителей', staff:0}
              : BUILDING_DEFS[c.id];
    const pct = (1 - c.remaining/c.total)*100;
    const speedCost = getSpeedUpCost(c);
    const canSpeed = S.money >= speedCost;
    const div=document.createElement('div');
    div.className='item';
    div.innerHTML=`
      <div class="item-head">
        <div style="flex:1">
          <div class="item-name">${def.name} <span class="build-time">${fmtTime(c.remaining)}</span></div>
          <div class="build-progress"><div class="build-progress-fill" style="width:${pct}%"></div></div>
        </div>
        <button class="speed" ${canSpeed?'':'disabled'} onclick="speedUpConstruction(${i})">
          ⚡<span class="cost">💰${fmt(speedCost)}</span>
        </button>
      </div>`;
    bEl.appendChild(div);
  });
  for (const [id,def] of Object.entries(BUILDING_DEFS)) {
    if (!isUnlocked(id)) continue;
    const owned=S.buildings[id]||0;
    const hasRes=canAffordResources(def.resCost);
    const can=S.money>=def.cost && hasRes && freeSlots>0 && (def.staff===0 || S.pop-totalStaffNeeded()-totalStaffInConstruction()>=def.staff);
    const resStr=def.resCost&&SETTINGS.showCosts ? Object.entries(def.resCost).map(([r,a])=>`${RES_DEFS[r].name.split(' ')[0]}${a}`).join(' ') : '';
    const div=document.createElement('div');
    div.className='item';
    div.innerHTML=`
      <div class="item-head">
        <div>
          <div class="item-name">${def.name} ${owned>0?`×${owned}`:''}</div>
          <div class="item-desc">${def.desc}${def.staff?` · 👷${def.staff}`:''} · ⏱️${fmtTime(def.buildTime)}</div>
        </div>
        <button ${can?'':'disabled'} onclick="build('${id}')">
          Строить<span class="cost">💰${fmt(def.cost)} ${resStr}</span>
        </button>
      </div>`;
    bEl.appendChild(div);
  }

  const gEl=document.getElementById('gathering');
  const prices={wood:50,stone:80,iron:200,gold:600};
  gEl.innerHTML=Object.entries(RES_DEFS).map(([res,def])=>{
    const have=Math.floor(S.resources[res]||0);
    const price=prices[res];
    return `<div class="item"><div class="item-head">
      <div><div class="item-name">${def.name}</div>
      <div class="item-desc">В наличии: ${fmt(have)} · ${price}/шт</div></div>
      <div style="display:flex;gap:4px;flex-direction:column">
        <button class="gold" ${have>=10?'':'disabled'} onclick="sellResource('${res}',10)">×10<span class


="cost">+${fmt(10*price)}</span></button>
        <button class="gold" ${have>=100?'':'disabled'} onclick="sellResource('${res}',100)">×100<span class="cost">+${fmt(100*price)}</span></button>
      </div>
    </div></div>`;
  }).join('');

  const aEl=document.getElementById('actions');
  const bh = BUILDER_HOUSE_COST;
  const canBH = S.money>=bh.money && canAffordResources({wood:bh.wood,stone:bh.stone}) && freeSlots>0;
  const acts=[
    {n:'🏗️ Построить дом строителей', d:`+${BUILDER_HOUSE_SLOTS} слота · всего: ${S.builderHouses}`, c:bh.money, res:`🪵${bh.wood} 🪨${bh.stone}`, fn:'buildBuilderHouse', dis:!canBH},
    {n:'🏚️ Восстановить дом', d:`+1 место · заброшено: ${S.abandoned}`, c:50000, res:'🪵30 🪨20', fn:'restoreHouse', dis:S.abandoned<=0||!canAffordResources({wood:30,stone:20})||freeSlots<=0},
    {n:'🧲 Привлечь жителя', d:`+1 житель · свободно: ${S.popCap-S.pop}`, c:20000, res:'', fn:'attractPeople', dis:S.popCap<=S.pop},
    {n:'💳 Выплатить кредит', d:`до 100K · осталось: ${fmt(S.loan)}`, c:0, res:'', fn:'payLoan', dis:S.loan<=0||S.money<1000}
  ];
  aEl.innerHTML=acts.map(a=>`
    <div class="item"><div class="item-head">
      <div><div class="item-name">${a.n}</div><div class="item-desc">${a.d}</div></div>
      <button class="${a.fn==='payLoan'?'gold':''}" ${a.dis?'disabled':''} onclick="${a.fn}()">
        ${a.c?`<span class="cost">💰${fmt(a.c)} ${a.res}</span>`:'Выполнить'}
      </button>
    </div></div>`).join('');
}

// ==================== СОХРАНЕНИЯ ====================
const SAVE_PREFIX = 'capital_v6_slot_';
const SAVE_SLOTS = 5;

function getSaveMeta(slot) {
  try {
    const d = localStorage.getItem(SAVE_PREFIX + slot);
    if (!d) return null;
    const p = JSON.parse(d);
    if (!p || !p.state) return null;
    return {
      slot,
      stage: p.state.stage,
      stageName: STAGES[p.state.stage]?.name || '?',
      money: p.state.money,
      pop: p.state.pop,
      day: p.state.day,
      builderHouses: p.state.builderHouses || 0,
      savedAt: p.savedAt
    };
  } catch(e) { return null; }
}
function saveToSlot(slot) {
  try {
    const data = { state: S, savedAt: Date.now() };
    localStorage.setItem(SAVE_PREFIX + slot, JSON.stringify(data));
    return true;
  } catch(e) { return false; }
}
function loadFromSlot(slot) {
  try {
    const d = localStorage.getItem(SAVE_PREFIX + slot);
    if (!d) return false;
    const p = JSON.parse(d);
    if (!p || !p.state) return false;
    S = { ...newGameState(), ...p.state };
    if (!S.construction) S.construction = [];
    if (!S.resources) S.resources = { wood:50, stone:30, iron:0, gold:0 };
    if (S.builderHouses === undefined) S.builderHouses = 1;
    const off = (Date.now() - (S.lastTick || Date.now()))/1000;
    if (off > 1) {
      const prod = getProduction();
      S.money += (prod.money + getTaxIncome()) * Math.min(off, 3600);
      S.food += (prod.food - getFoodConsumption()) * Math.min(off, 3600);
      const g = getGathering();
      for (const [r,a] of Object.entries(g)) S.resources[r] = (S.resources[r]||0) + a*Math.min(off,3600);
      S.construction.forEach(c => { c.remaining -= Math.min(off, 3600); });
      const done = S.construction.filter(c => c.remaining <= 0);
      S.construction = S.construction.filter(c => c.remaining > 0);
      done.forEach(c => {
        if (c.id === 'house_restore') { S.abandoned--; S.houses++; S.popCap++; }
        else if (c.id === 'builder_house') { S.builderHouses++; }
        else S.buildings[c.id] = (S.buildings[c.id]||0) + 1;
      });
    }
    S.lastTick = Date.now();
    return true;
  } catch(e) { return false; }
}
function deleteSlot(slot) {
  try { localStorage.removeItem(SAVE_PREFIX + slot); return true; } catch(e){ return false; }
}
function save() {
  if (!gameRunning) return;
  saveToSlot(0);
}

function openSavesModal() {
  const list = document.getElementById('savesList');
  let html = '';
  for (let i = 0; i < SAVE_SLOTS; i++) {
    const meta = getSaveMeta(i);
    const slotN


ame = i === 0 ? 'Быстрое сохранение' : `Слот ${i}`;
    if (meta) {
      const date = new Date(meta.savedAt).toLocaleString('ru-RU');
      html += `<div class="save-slot">
        <div class="save-info">
          <div class="save-name">${slotName} · ${meta.stageName}</div>
          <div class="save-meta">💰${fmt(meta.money)} · 👥${meta.pop} · 🏗️${meta.builderHouses||0} · День ${meta.day} · ${date}</div>
        </div>
        <div class="save-actions">
          <button onclick="loadSlotAction(${i})">Загрузить</button>
          <button onclick="overwriteSlot(${i})">Перезаписать</button>
          <button class="del" onclick="deleteSlotAction(${i})">✕</button>
        </div>
      </div>`;
    } else {
      html += `<div class="save-slot empty">
        <div class="save-info">
          <div class="save-name">${slotName}</div>
          <div class="save-meta">Пусто</div>
        </div>
        <div class="save-actions">
          <button onclick="saveToSlotAction(${i})">Сохранить</button>
        </div>
      </div>`;
    }
  }
  list.innerHTML = html;
  document.getElementById('savesModal').classList.add('show');
}
function loadSlotAction(slot) {
  if (gameRunning && !confirm('Загрузить сохранение? Текущий прогресс будет потерян (если не сохранён).')) return;
  if (loadFromSlot(slot)) {
    document.getElementById('savesModal').classList.remove('show');
    startGameWithState();
    toast('📂 Сохранение загружено');
    playClick();
  } else {
    alert('Не удалось загрузить сохранение');
  }
}
function saveToSlotAction(slot) {
  if (!gameRunning) { alert('Сначала начните игру'); return; }
  if (saveToSlot(slot)) {
    toast('💾 Сохранено в слот '+slot);
    openSavesModal();
    playClick();
  }
}
function overwriteSlot(slot) {
  if (!confirm('Перезаписать слот '+slot+'?')) return;
  if (saveToSlot(slot)) {
    toast('💾 Перезаписано');
    openSavesModal();
  }
}
function deleteSlotAction(slot) {
  if (!confirm('Удалить сохранение из слота '+slot+'?')) return;
  deleteSlot(slot);
  openSavesModal();
}

// ==================== ГЛАВНОЕ МЕНЮ ====================
function menuStartGame() {
  if (getSaveMeta(0)) {
    if (!confirm('Начать новую игру? Быстрое сохранение будет перезаписано при первом сохранении.')) return;
  }
  S = newGameState();
  S.lastTick = Date.now();
  startGameWithState();
  addLog('1 000 000 в кредит. Развивай деревню!','warn');
  addLog('🏗️ У тебя 1 дом строителей = 4 слота строительства.','warn');
  addLog('⚡ Стройку можно ускорить: 10 сек = 200 денег.','warn');
}
function menuLoadSave() { openSavesModal(); }
function menuExit() {
  if (confirm('Выйти из игры? Несохранённый прогресс будет потерян.')) {
    stopMusic();
    gameRunning = false;
    document.getElementById('gameWrap').style.display = 'none';
    document.getElementById('mainMenu').classList.remove('hidden');
    try { window.close(); } catch(e){}
  }
}
function startGameWithState() {
  document.getElementById('mainMenu').classList.add('hidden');
  document.getElementById('gameWrap').style.display = 'block';
  gameRunning = true;
  S.lastTick = Date.now();
  resizeCanvas();
  document.getElementById('taxRate').value = S.taxRate;
  document.getElementById('taxVal').textContent = S.taxRate+'%';
  render();
  updateClockUI();
  drawMap();
  initAudio();
  updateMusic();
}
function backToMenu() {
  if (!confirm('Вернуться в главное меню? Прогресс сохранится в быстрый слот.')) return;
  save();
  stopMusic();
  gameRunning = false;
  document.getElementById('gameWrap').style.display = 'none';
  document.getElementById('mainMenu').classList.remove('hidden');
}

// ==================== СТАРТ ====================
loadSettings();
resizeCanvas();
setTimeout(() => { resizeCanvas(); syncSettingsUI(); }, 100);
document.addEventListener('click', () => {
  initAudio();
  if (gameRunning) updateMusic();
}, { once: true });
</script>
</body>
</html>

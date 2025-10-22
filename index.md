<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>IN • ON • AT — тренажёр (v3)</title>
  <style>
    html, body { margin: 0; padding: 0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background: #0f1220; color: #e9ecf1; }
    .wrap { max-width: 960px; margin: 0 auto; padding: 24px; }
    h1 { font-size: 28px; margin: 0 0 8px; letter-spacing: 0.3px; }
    .sub { color: #b7c0d1; margin-bottom: 16px; }
    .card { background: #171a2b; border: 1px solid #272b44; border-radius: 14px; padding: 18px; margin: 12px 0; }
    .q { font-size: 18px; line-height: 1.45; }
    .blank { display: inline-block; min-width: 48px; text-align: center; padding: 2px 8px; border-bottom: 2px dashed #596080; margin: 0 6px; }
    .btns { display: flex; gap: 10px; margin-top: 10px; flex-wrap: wrap; }
    button.choice { border: 1px solid #3a4062; background: #1e2340; color: #e9ecf1; padding: 10px 16px; border-radius: 12px; font-size: 16px; cursor: pointer; }
    button.choice:hover { border-color: #6f78ff; }
    button.choice.correct { background: #123d28; border-color: #1aae67; }
    button.choice.wrong { background: #4a1e2a; border-color: #ff557a; }
    .answer-line { margin-top: 10px; display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
    input.free { background: #0e1830; border: 1px solid #2d3353; color: #e9ecf1; border-radius: 10px; padding: 8px 12px; font-size: 16px; width: 150px; }
    .exp { margin-top: 10px; color: #c7d1ea; font-size: 14px; display: none; }
    .topbar { display: flex; justify-content: space-between; align-items: center; gap: 10px; flex-wrap: wrap; margin: 10px 0 16px; }
    .pill { border: 1px solid #303659; padding: 8px 12px; border-radius: 999px; color: #d9def0; background: #14192e; cursor: pointer; }
    .stats { display: flex; gap: 12px; color: #c9d4ee; flex-wrap: wrap; }
    .actions { display: flex; gap: 10px; flex-wrap: wrap; }
    .actions button { border: 1px solid #3a4062; background: #131935; color: #e9ecf1; padding: 8px 12px; border-radius: 10px; cursor: pointer; }
    .actions button:hover { border-color: #6f78ff; }
    .cat { font-size: 12px; color: #a1aacb; margin-left: 6px; }
    .hidden { display: none; }
    .banner { background: #2a2139; border: 1px dashed #5b5681; margin-bottom: 12px; padding: 10px 12px; border-radius: 10px; color: #d9d3ff; }
    .list { display: grid; grid-template-columns: 1fr; gap: 10px; }
    @media (min-width: 700px){
      .list { grid-template-columns: 1fr 1fr; }
    }
    .badge { padding: 2px 8px; border-radius: 999px; border:1px solid #3a4062; font-size: 12px; color:#cbd3f3; }
  </style>
</head>
<body>
<div class="wrap">
  <h1>IN • ON • AT — интерактивный тренажёр <span class="badge">v3</span></h1>
  <div class="sub">Выбирай или вписывай ответ. Можно менять вариант — карточка засчитывается только когда ответ станет верным.</div>
  <div id="js-warning" class="banner hidden">
    Если здесь «Прогресс: 0/0», браузер блокирует скрипты. На iPhone: «Поделиться» → <b>Открыть в Safari</b>.
  </div>

  <div class="topbar">
    <div class="actions">
      <button id="shuffle">Перемешать</button>
      <button id="reset">Сбросить результаты</button>
      <button id="review">Повторить ошибки</button>
      <button id="export">Экспорт статистики</button>
      <button id="show-all-explanations">Показать объяснения</button>
      <button id="hide-all-explanations" class="hidden">Скрыть объяснения</button>
    </div>
    <div class="stats">
      <div>Прогресс: <span id="done">0</span>/<span id="total">0</span></div>
      <div>Верных: <span id="right">0</span></div>
    </div>
  </div>

  <label class="pill"><input type="checkbox" id="freeToggle"/> Режим ввода ответа вручную</label>

  <div class="list" id="list"></div>
</div>

<script>
const STORAGE_KEY = 'in-on-at-stats-v3';

const ITEMS = [
  { s: "The meeting starts ___ 3 p.m.", a: ["at"], cat:"time", exp:"Точное время → <b>at</b>." },
  { s: "We have classes ___ Monday.", a: ["on"], cat:"time", exp:"Дни недели → <b>on</b>." },
  { s: "Their wedding is ___ June.", a: ["in"], cat:"time", exp:"Месяцы → <b>in</b>." },
  { s: "He was born ___ 1999.", a: ["in"], cat:"time", exp:"Годы → <b>in</b>." },
  { s: "The store closes ___ night.", a: ["at"], cat:"time", exp:"Исключение: <b>at night</b>." },
  { s: "We usually go to the gym ___ the morning.", a: ["in"], cat:"time", exp:"Части суток → <b>in the morning</b>." },
  { s: "The party is ___ Friday evening.", a: ["on"], cat:"time", exp:"Дни + части суток → <b>on</b>." },
  { s: "I will see you ___ the weekend.", a: ["at","on"], cat:"time", exp:"BrE: <b>at</b>, AmE: <b>on</b> the weekend." },
  { s: "Our exam is ___ the 15th of May.", a: ["on"], cat:"time", exp:"Конкретные даты → <b>on</b>." },
  { s: "They moved here ___ spring.", a: ["in"], cat:"time", exp:"Сезоны → <b>in</b>." },
  { s: "She is waiting ___ the bus stop.", a: ["at"], cat:"place", exp:"Точка/место встречи → <b>at</b>." },
  { s: "He lives ___ London.", a: ["in"], cat:"place", exp:"Города/страны → <b>in</b>." },
  { s: "The book is ___ the table.", a: ["on"], cat:"place", exp:"Поверхность → <b>on</b>." },
  { s: "I left my keys ___ my pocket.", a: ["in"], cat:"place", exp:"Внутри чего-то → <b>in</b>." },
  { s: "We met ___ the airport.", a: ["at"], cat:"place", exp:"Точки/терминалы → <b>at</b>." },
  { s: "They’re swimming ___ the river.", a: ["in"], cat:"place", exp:"Вода/объём → <b>in</b>." },
  { s: "There’s a picture ___ the wall.", a: ["on"], cat:"place", exp:"Вертикальная поверхность → <b>on</b>." },
  { s: "Let’s meet ___ the corner of the street.", a: ["at","on"], cat:"place", exp:"Обычно <b>at</b>, в BrE также <b>on</b>." },
  { s: "We arrived ___ the hotel at 6.", a: ["at"], cat:"place", exp:"Arrive at (точка)." },
  { s: "We arrived ___ Paris at noon.", a: ["in"], cat:"place", exp:"Arrive in (город/страна)." },
  { s: "He is ___ the bus.", a: ["on"], cat:"transport", exp:"Общ. транспорт/борт → <b>on</b>." },
  { s: "She is ___ a taxi.", a: ["in"], cat:"transport", exp:"Малый транспорт → <b>in</b>." },
  { s: "We are ___ a car.", a: ["in"], cat:"transport", exp:"Внутри машины → <b>in</b>." },
  { s: "They’re traveling ___ a plane right now.", a: ["on"], cat:"transport", exp:"На борту → <b>on</b>." },
  { s: "I go to work ___ foot.", a: ["on"], cat:"transport", exp:"Идиома: <b>on foot</b>." },
  { s: "She lives ___ Baker Street.", a: ["on"], cat:"addresses", exp:"Улицы (AmE) → <b>on</b>." },
  { s: "The office is ___ 27 King Road.", a: ["at"], cat:"addresses", exp:"Полный адрес с номером → <b>at</b>." },
  { s: "I saw it ___ TV.", a: ["on"], cat:"addresses", exp:"Медиа → <b>on TV</b>." },
  { s: "He’s ___ the phone right now.", a: ["on"], cat:"addresses", exp:"Идиома → <b>on the phone</b>." },
  { s: "I’m ___ home.", a: ["at"], cat:"addresses", exp:"Фикс. выражение → <b>at home</b>." },
  { s: "She’s ___ work.", a: ["at"], cat:"addresses", exp:"Фикс. выражение → <b>at work</b>." },
  { s: "The kids are ___ school.", a: ["at","in"], cat:"addresses", exp:"Цель/занятия → <b>at</b>; физически внутри → <b>in</b> (AmE)." },
  { s: "Birds are flying ___ the sky.", a: ["in"], cat:"addresses", exp:"Объёмная среда → <b>in</b>." },
  { s: "There’s a scratch ___ the screen.", a: ["on"], cat:"addresses", exp:"Поверхность → <b>on</b>." },
  { s: "We’ll stop ___ the way.", a: ["on"], cat:"addresses", exp:"Идиома → <b>on the way</b>." },
  { s: "He’s standing ___ the door.", a: ["at","in"], cat:"place", exp:"У двери → <b>at</b>; в проёме → <b>in</b>." },
  { s: "I left a note ___ your desk.", a: ["on"], cat:"place", exp:"Поверхность стола → <b>on</b>." },
  { s: "There’s a meeting ___ Room 402.", a: ["in","at"], cat:"place", exp:"Обычно <b>in</b>, как точка на схеме — <b>at</b>." },
  { s: "She’s ___ the beach now.", a: ["at","on"], cat:"place", exp:"Локация → <b>at</b>; поверхность песка → <b>on</b>." },
  { s: "The picture is ___ the cover of the magazine.", a: ["on"], cat:"place", exp:"Обложка → <b>on</b>." },
  { s: "We stayed ___ a small island.", a: ["on"], cat:"place", exp:"Остров → <b>on</b>." },
  { s: "He’s ___ the hospital visiting his friend.", a: ["at","in"], cat:"place", exp:"Учреждение/цель → <b>at</b>; физически внутри → <b>in</b>." },
  { s: "The show starts ___ noon.", a: ["at"], cat:"time", exp:"Полдень/полночь → <b>at</b>." }
];

const list = document.getElementById('list');
const totalEl = document.getElementById('total');
const doneEl  = document.getElementById('done');
const rightEl = document.getElementById('right');
const shuffleBtn = document.getElementById('shuffle');
const resetBtn = document.getElementById('reset');
const freeToggle = document.getElementById('freeToggle');
const showAllBtn = document.getElementById('show-all-explanations');
const hideAllBtn = document.getElementById('hide-all-explanations');
const reviewBtn = document.getElementById('review');
const exportBtn = document.getElementById('export');
const jsWarn = document.getElementById('js-warning');

let DATA = ITEMS.slice();
let correctCount = 0;
let STATS = loadStats();

function saveStats(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(STATS)); }
function loadStats(){
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if(!raw) return { perItem: {}, totals: {attempts:0, correct:0} };
    return JSON.parse(raw);
  } catch(e){
    return { perItem: {}, totals: {attempts:0, correct:0} };
  }
}
function shuffleArray(arr){
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
}
function render(){
  list.innerHTML = '';
  DATA.forEach((item, idx) => {
    const card = document.createElement('div');
    card.className = 'card';
    card.dataset.index = idx;

    const q = document.createElement('div');
    q.className = 'q';
    q.innerHTML = item.s.replace('___', `<span class="blank" aria-label="пустое место">___</span>`)
                     + ` <span class="cat">(${item.cat})</span>`;
    card.appendChild(q);

    const btns = document.createElement('div');
    btns.className = 'btns';
    ['in','on','at'].forEach(p => {
      const b = document.createElement('button');
      b.className = 'choice';
      b.textContent = p.toUpperCase();
      b.addEventListener('click', () => check(idx, p, b, 'button'));
      btns.appendChild(b);
    });
    card.appendChild(btns);

    const ansLine = document.createElement('div');
    ansLine.className = 'answer-line';
    const input = document.createElement('input');
    input.className = 'free';
    input.placeholder = 'впиши: in / on / at';
    input.setAttribute('inputmode', 'latin');
    const checkBtn = document.createElement('button');
    checkBtn.textContent = 'Проверить';
    checkBtn.addEventListener('click', () => check(idx, input.value.trim().toLowerCase(), input, 'input'));
    ansLine.appendChild(input);
    ansLine.appendChild(checkBtn);
    card.appendChild(ansLine);

    const exp = document.createElement('div');
    exp.className = 'exp';
    exp.innerHTML = `Объяснение: ${item.exp}`;
    card.appendChild(exp);

    list.appendChild(card);
  });
  totalEl.textContent = DATA.length;
  applyFreeMode();
  jsWarn.classList.add('hidden');
  updateCounters();
}
function applyFreeMode(){
  const manual = freeToggle.checked;
  document.querySelectorAll('.btns').forEach(el => el.style.display = manual ? 'none' : 'flex');
  document.querySelectorAll('.answer-line').forEach(el => el.style.display = manual ? 'flex' : 'none');
}
function updateCounters(){
  doneEl.textContent = String(correctCount);
  rightEl.textContent = String(correctCount);
}
function incStats(ok, absoluteIndex){
  STATS.totals.attempts += 1;
  if(ok) STATS.totals.correct += 1;
  const slot = STATS.perItem[absoluteIndex] || { attempts:0, correct:0, wrong:0 };
  slot.attempts += 1;
  if(ok) slot.correct += 1; else slot.wrong += 1;
  STATS.perItem[absoluteIndex] = slot;
  saveStats();
}
function check(idx, val, el, source){
  const card = list.querySelector(`.card[data-index="${idx}"]`);
  const answers = DATA[idx].a;
  const exp = card.querySelector('.exp');
  const blankSpan = card.querySelector('.blank');

  let v = (val || '').toLowerCase();
  if(!['in','on','at'].includes(v)){
    if (freeToggle.checked) {
      exp.style.display = 'block';
      exp.innerHTML = 'Подсказка: используй только <b>in</b>, <b>on</b> или <b>at</b>.';
    }
    return;
  }

  // Clear previous highlights in this card
  card.querySelectorAll('.choice').forEach(b => b.classList.remove('correct','wrong'));

  const isCorrect = answers.includes(v);
  if(source === 'button' && el && el.classList){
    el.classList.add(isCorrect ? 'correct' : 'wrong');
  }
  blankSpan.textContent = v.toUpperCase();

  // Fresh explanation each time
  exp.style.display = 'block';
  exp.innerHTML = (isCorrect
    ? 'Верно! '
    : 'Не совсем. Правильно: <b>' + answers.join('</b> / <b>') + '</b>. '
  ) + DATA[idx].exp;

  // Stats
  const absoluteIndex = ITEMS.findIndex(it => it.s === DATA[idx].s);
  incStats(isCorrect, absoluteIndex);

  // Count only first correct
  if(isCorrect && card.dataset.correct !== '1'){
    card.dataset.correct = '1';
    correctCount += 1;
    updateCounters();
  }
}
function resetStatsUI(){
  correctCount = 0;
  document.querySelectorAll('.card').forEach(c => {
    c.removeAttribute('data-correct');
    c.querySelectorAll('.choice').forEach(b => b.classList.remove('correct','wrong'));
    c.querySelector('.exp').style.display = 'none';
    c.querySelector('.blank').textContent = '___';
    const input = c.querySelector('input.free');
    if (input) input.value = '';
  });
  updateCounters();
}
function showAllExplanations(show){
  document.querySelectorAll('.exp').forEach(e => e.style.display = show ? 'block' : 'none');
  showAllBtn.classList.toggle('hidden', show);
  hideAllBtn.classList.toggle('hidden', !show);
}
function reviewMistakes(){
  const wrongList = Object.entries(STATS.perItem)
    .filter(([i,v]) => v.wrong > 0)
    .sort((a,b) => b[1].wrong - a[1].wrong)
    .map(([i]) => ITEMS[Number(i)]);
  if(wrongList.length === 0){
    alert('Отлично! Нет накопленных ошибок.');
    return;
  }
  DATA = wrongList.slice();
  render();
  resetStatsUI();
}
function exportStats(){
  const perCat = {};
  Object.entries(STATS.perItem).forEach(([i, v]) => {
    const cat = ITEMS[i].cat;
    perCat[cat] = perCat[cat] || { attempts:0, correct:0, wrong:0 };
    perCat[cat].attempts += v.attempts;
    perCat[cat].correct += v.correct;
    perCat[cat].wrong += v.wrong;
  });
  const payload = {
    totals: STATS.totals,
    perCategory: perCat,
    hardestItems: Object.entries(STATS.perItem)
      .sort((a,b)=>b[1].wrong - a[1].wrong)
      .slice(0,10)
      .map(([i,v]) => ({ sentence: ITEMS[i].s, answers: ITEMS[i].a, stats: v }))
  };
  const txt = JSON.stringify(payload, null, 2);
  navigator.clipboard?.writeText(txt).then(
    ()=>alert('Статистика скопирована. Вставь её мне в чат — сделаю персональный сет.'),
    ()=>alert('Скопировать не вышло. Появится окно — выдели и скопируй вручную.\n\n' + txt)
  );
}
function init(){
  try{
    shuffleArray(DATA);
    render();
    resetStatsUI();
  }catch(e){
    document.getElementById('js-warning').classList.remove('hidden');
  }
}
document.getElementById('shuffle').addEventListener('click', () => { DATA = ITEMS.slice(); shuffleArray(DATA); render(); resetStatsUI(); });
document.getElementById('reset').addEventListener('click', resetStatsUI);
freeToggle.addEventListener('change', applyFreeMode);
showAllBtn.addEventListener('click', () => showAllExplanations(true));
hideAllBtn.addEventListener('click', () => showAllExplanations(false));
document.getElementById('review').addEventListener('click', reviewMistakes);
document.getElementById('export').addEventListener('click', exportStats);

init();
</script>
</body>
</html>

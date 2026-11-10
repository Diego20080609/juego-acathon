<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>El Depurador de Código · Mazmorra de la Nube</title>
<style>
:root{--bg:#0a0f18;--card:#131b2a;--tx:#e8edf5;--mut:#8593a8;--ac:#c8f169;--bad:#ff6b6b;--warn:#ffc84a;--ln:#243049}
*{box-sizing:border-box}
body{margin:0 auto;max-width:760px;min-height:100vh;padding:14px;color:var(--tx);font:16px/1.45 system-ui,"Segoe UI",sans-serif;background:radial-gradient(900px 380px at 50% -120px,#1b2b4a,var(--bg)) var(--bg)}
[hidden]{display:none!important}
h1{font-size:1.7rem;line-height:1.15;margin:0 0 8px}
h2{font-size:.9rem;color:var(--mut);font-weight:600;margin:0 0 8px}
.top{display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:12px;font-size:.9rem;color:var(--mut)}
.top b{color:var(--tx)}
.hp{color:var(--bad);letter-spacing:2px}
.card{background:var(--card);border:1px solid var(--ln);border-radius:14px;padding:14px;margin-bottom:12px}
.mono,#code,table,#out{font-family:ui-monospace,Consolas,"Courier New",monospace}
.hint{font-size:.85rem;color:var(--mut);margin:10px 0 0}
#code{font-size:.95rem;overflow-x:auto}
.ln{white-space:pre;padding:6px 8px;border-radius:8px;border-left:3px solid transparent;cursor:pointer;transition:opacity .2s}
.ln:hover{background:#1a2438}
.ln i{display:inline-block;width:2.2ch;margin-right:12px;color:var(--mut);font-style:normal;text-align:center;border-radius:50%}
.ln i.on{background:var(--bad);color:#fff}
.ln.cur{background:#3a3210;border-left-color:var(--warn)}
.ln.ok{background:#1d3a22;border-left-color:var(--ac)}
.ln.bad{background:#3a1a1e;border-left-color:var(--bad)}
.ln.dim{opacity:.3}
table{width:100%;border-collapse:collapse}
tr{cursor:pointer}
td{padding:6px 8px;border-bottom:1px solid var(--ln)}
td:first-child{color:var(--mut);width:38%}
tr.chg td{color:var(--warn);font-weight:700;animation:fl .7s}
tr.pin td:first-child{color:var(--ac)}
tr.pin td:first-child::before{content:"📌 "}
@keyframes fl{from{background:#5a4a14}to{background:transparent}}
#hist{margin-top:8px;font-size:.9rem;color:var(--mut)}
#hist b{color:var(--ac)}
#out{margin-top:10px}
.good{color:var(--ac)}.err{color:var(--bad)}
input[type=range]{width:100%;accent-color:var(--ac);margin:4px 0 10px}
input[type=text],select{padding:10px;border-radius:10px;border:1px solid var(--ln);background:#0d1421;color:var(--tx);font-size:1rem}
input[type=text]{width:100%;margin-bottom:12px}
.btns{display:flex;flex-wrap:wrap;gap:8px}
button{background:var(--ln);color:var(--tx);border:0;border-radius:10px;padding:11px 14px;font-size:.95rem;cursor:pointer}
button.main{background:var(--ac);color:#10200a;font-weight:700}
button:disabled{opacity:.35;cursor:default}
button:focus-visible,input:focus-visible,select:focus-visible{outline:2px solid var(--ac);outline-offset:2px}
.opt{display:block;width:100%;text-align:left;margin-bottom:8px;font-family:ui-monospace,Consolas,monospace;white-space:pre-wrap}
#msg{min-height:1.5em;margin:0 0 12px}
.big{font-size:2.2rem;font-weight:800;color:var(--ac);margin:6px 0}
.hit{animation:sh .3s}
@keyframes sh{25%{transform:translateX(-6px)}75%{transform:translateX(6px)}}
.pt{position:fixed;top:55vh;width:8px;height:8px;border-radius:2px;pointer-events:none;animation:pt 1s ease-out forwards}
@keyframes pt{to{transform:translate(var(--dx),calc(var(--dy) + 260px)) rotate(540deg);opacity:0}}
@media (prefers-reduced-motion:reduce){.hit,tr.chg td,.pt{animation:none}.ln{transition:none}}
</style>
</head>
<body>

<div class="top">
  <span id="lvl">La Mazmorra de la Nube</span>
  <span class="hp" id="hp"></span>
  <span>Puntos <b id="score">0</b></span>
  <span>⏱ <b id="time">--</b></span>
</div>

<div class="card" id="start">
  <h1>El Depurador de Código</h1>
  <p>Los servidores de la Mazmorra de la Nube ejecutan programas dañados. Cada uno imprime un resultado incorrecto. Investiga, encuentra la instrucción culpable y elige la corrección.</p>
  <p class="mono" style="font-size:.88rem;color:var(--mut)">
    • Predice la salida antes de ejecutar (+20).<br>
    • Avanza paso a paso, arrastra la línea de tiempo o usa ▶ Auto.<br>
    • Toca el número de una línea: punto de parada.<br>
    • Toca una variable: la vigilas y ves su historial.<br>
    • Toca la línea culpable y luego elige la corrección.<br>
    • 5 vidas, 3 pisos: el tiempo baja en cada piso.
  </p>
  <input type="text" id="nm" maxlength="14" placeholder="Tu nombre para el ranking" aria-label="Tu nombre">
  <div class="btns">
    <button class="main" onclick="startGame()">Iniciar investigación</button>
    <button onclick="toggleSound(this)">Sonido: sí</button>
  </div>
  <p class="hint">Atajos: ← → avanzan o retroceden, C continúa hasta el siguiente punto de parada, Espacio = Auto.</p>
</div>

<div id="game" hidden>
  <div class="card" id="goal"></div>
  <div class="card" id="pred" hidden></div>
  <div class="card"><h2>Programa · toca una línea para acusarla</h2><div id="code"></div></div>
  <div class="card">
    <h2>Memoria (variables) · toca una para vigilarla</h2>
    <table id="vars"></table>
    <div id="hist"></div>
    <div id="out"></div>
  </div>
  <div class="card" id="ctl">
    <input type="range" id="sl" min="-1" value="-1" oninput="go(+this.value)" aria-label="Línea de tiempo de la ejecución">
    <div class="btns">
      <button id="bBack" onclick="go(S.st-1)">◀ Atrás</button>
      <button id="bStep" class="main" onclick="go(S.st+1)">Paso ▶</button>
      <button id="bAuto" onclick="toggleAuto()">▶ Auto</button>
      <select id="spd" aria-label="Velocidad"><option value="900">Lento</option><option value="500" selected>Normal</option><option value="220">Rápido</option></select>
      <button id="bCont" onclick="cont()">Continuar ▶▶</button>
      <button id="bAll" onclick="go(S.tr.length-1)">Ejecutar todo</button>
      <button onclick="go(-1)">Reiniciar</button>
      <button id="bScan" onclick="scan()">Escáner (-20)</button>
      <button id="bHint" onclick="hint()">Pista (-15)</button>
    </div>
  </div>
  <div class="card" id="fix" hidden></div>
  <p id="msg" role="status"></p>
  <button class="main" id="bNext" onclick="nextCase()" hidden>Siguiente caso ▶</button>
</div>

<div class="card" id="endscr" hidden></div>

<script>
/* CASOS. v guarda las variables y T(n) registra un paso en la línea n.
   sus = líneas sospechosas que deja visibles el Escáner.
   fix[0] es la corrección correcta (luego se mezclan). */
const L = [
  { t: "La suma incompleta", goal: "Debe imprimir <b>10</b> (1+2+3+4).",
    code: ["total = 0", "for i in range(1, 4):", "    total += i", "print(total)"],
    run: (v, T) => { v.total = 0; T(0); for (const i of [1, 2, 3]) { v.i = i; T(1); v.total += i; T(2); } T(3); },
    out: "6", exp: "10", bug: 1, sus: [1, 2],
    hint: "Cuenta cuántas veces cambia la variable i.",
    fix: ["for i in range(1, 5):", "for i in range(0, 4):", "for i in range(1, 3):"],
    why: "range(1, 4) se detiene en 3 y deja fuera el 4." },

  { t: "El promedio falso", goal: "Debe imprimir <b>6.0</b> (promedio de 4, 8 y 6).",
    code: ["notas = [4, 8, 6]", "suma = 0", "for n in notas:", "    suma += n", "promedio = suma / 2", "print(promedio)"],
    run: (v, T) => { v.notas = [4, 8, 6]; T(0); v.suma = 0; T(1);
      for (const n of v.notas) { v.n = n; T(2); v.suma += n; T(3); }
      v.promedio = (v.suma / 2).toFixed(1); T(4); T(5); },
    out: "9.0", exp: "6.0", bug: 4, sus: [3, 4],
    hint: "La suma es correcta (18). ¿Entre cuánto se divide?",
    fix: ["promedio = suma / len(notas)", "promedio = suma / 4", "promedio = suma * 2"],
    why: "Dividir entre 2 es un número fijo; son 3 notas, se usa len(notas)." },

  { t: "El factorial que da cero", goal: "Debe imprimir <b>24</b> (4!).",
    code: ["resultado = 0", "for i in range(1, 5):", "    resultado *= i", "print(resultado)"],
    run: (v, T) => { v.resultado = 0; T(0); for (const i of [1, 2, 3, 4]) { v.i = i; T(1); v.resultado *= i; T(2); } T(3); },
    out: "0", exp: "24", bug: 0, sus: [0, 2],
    hint: "Mira el valor de resultado desde el primer paso.",
    fix: ["resultado = 1", "resultado = 2", "for i in range(0, 5):"],
    why: "Todo número por 0 es 0. El producto debe empezar en 1." },

  { t: "El descuento que no descuenta", goal: "Debe imprimir <b>80.0</b> (20% de descuento sobre 100).",
    code: ["precio = 100", "descuento = 20", "precio = precio - descuento / 100", "print(precio)"],
    run: (v, T) => { v.precio = 100; T(0); v.descuento = 20; T(1);
      v.precio = (v.precio - v.descuento / 100).toFixed(1); T(2); T(3); },
    out: "99.8", exp: "80.0", bug: 2, sus: [1, 2],
    hint: "Solo se restan 0.2, no el 20% del precio.",
    fix: ["precio = precio - precio * descuento / 100", "precio = precio - descuento", "precio = precio / descuento"],
    why: "Se resta descuento/100 en lugar del porcentaje del precio." },

  { t: "El mayor que resultó menor", goal: "Debe imprimir <b>9</b> (el mayor de la lista).",
    code: ["nums = [3, 9, 2]", "mayor = nums[0]", "for n in nums:", "    if n < mayor:", "        mayor = n", "print(mayor)"],
    run: (v, T) => { v.nums = [3, 9, 2]; T(0); v.mayor = 3; T(1);
      for (const n of v.nums) { v.n = n; T(2); T(3); if (n < v.mayor) { v.mayor = n; T(4); } } T(5); },
    out: "2", exp: "9", bug: 3, sus: [1, 3],
    hint: "Cuando n vale 9, ¿se actualiza mayor? ¿Y cuando vale 2?",
    fix: ["if n > mayor:", "if n == mayor:", "if n <= 0:"],
    why: "La comparación está invertida: guarda el menor en vez del mayor." },

  { t: "Pares que no son pares", goal: "Debe imprimir <b>2</b> (hay dos pares: 4 y 10).",
    code: ["cont = 0", "for n in [4, 7, 10, 3, 5]:", "    if n % 2 == 1:", "        cont += 1", "print(cont)"],
    run: (v, T) => { v.cont = 0; T(0);
      for (const n of [4, 7, 10, 3, 5]) { v.n = n; T(1); T(2); if (n % 2 == 1) { v.cont++; T(3); } } T(4); },
    out: "3", exp: "2", bug: 2, sus: [2, 3],
    hint: "¿Qué números hicieron subir cont?",
    fix: ["if n % 2 == 0:", "if n % 2 == 2:", "if n // 2 == 1:"],
    why: "n % 2 == 1 detecta impares. Los pares dan residuo 0." },

  { t: "Una vuelta de más", goal: "Con 100 de saldo y retiros de 30, debe imprimir <b>4</b> retiros.",
    code: ["saldo = 100", "veces = 1", "while saldo > 0:", "    saldo -= 30", "    veces += 1", "print(veces)"],
    run: (v, T) => { v.saldo = 100; T(0); v.veces = 1; T(1);
      while (v.saldo > 0) { T(2); v.saldo -= 30; T(3); v.veces++; T(4); } T(5); },
    out: "5", exp: "4", bug: 1, sus: [1, 3],
    hint: "Compara veces con las vueltas reales del while.",
    fix: ["veces = 0", "veces = -1", "saldo = 90"],
    why: "El contador debe empezar en 0: aún no se hizo ningún retiro." },

  { t: "El acumulador amnésico", goal: "Debe imprimir <b>6</b> (1+2+3).",
    code: ["for i in range(1, 4):", "    total = 0", "    total += i", "print(total)"],
    run: (v, T) => { for (const i of [1, 2, 3]) { v.i = i; T(0); v.total = 0; T(1); v.total += i; T(2); } T(3); },
    out: "3", exp: "6", bug: 1, sus: [1, 2],
    hint: "Observa qué pasa con total al empezar cada vuelta.",
    fix: ["Mover total = 0 antes del for", "Cambiar total = 0 por total = 1", "Quitar la línea total += i"],
    why: "Reiniciar total en cada vuelta borra lo acumulado. Se inicializa una vez, antes del ciclo." },

  { t: "La función que triplica", goal: "<b>doble(4)</b> debe imprimir <b>8</b>.",
    code: ["def doble(x):", "    resultado = x * 3", "    return resultado", "print(doble(4))"],
    run: (v, T) => { T(3); v.x = 4; T(0); v.resultado = 12; T(1); T(2); T(3); },
    out: "12", exp: "8", bug: 1, sus: [1, 2],
    hint: "Compara el nombre de la función con lo que calcula.",
    fix: ["resultado = x * 2", "resultado = x + 2", "return x"],
    why: "La función se llama doble pero multiplica por 3." },

  { t: "El intercambio roto", goal: "Debe intercambiar los valores: imprimir <b>8 5</b>.",
    code: ["a = 5", "b = 8", "a = b", "b = a", "print(a, b)"],
    run: (v, T) => { v.a = 5; T(0); v.b = 8; T(1); v.a = v.b; T(2); v.b = v.a; T(3); T(4); },
    out: "8 8", exp: "8 5", bug: 2, sus: [2, 3],
    hint: "Después de a = b, ¿todavía existe el valor original de a?",
    fix: ["Guardar temp = a antes de a = b", "Cambiar b = a por b = b", "Cambiar a = b por a = a"],
    why: "a = b pisa el 5 antes de guardarlo. Hace falta una variable temporal." }
];

/* ESTADO Y UTILIDADES */
const TIMES = [90, 75, 60];
const $ = id => document.getElementById(id);
const esc = s => String(s).replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
const fmt = x => Array.isArray(x) ? "[" + x.join(", ") + "]" : String(x);
const floorOf = () => S.li < 3 ? 0 : S.li < 7 ? 1 : 2;
let S = {}, ac, mute = false, timer, auto = null;

function snd(f, d = .12, ty = "sine") {
  if (mute) return;
  try {
    ac = ac || new AudioContext();
    const o = ac.createOscillator(), g = ac.createGain();
    o.type = ty; o.frequency.value = f; g.gain.value = .05;
    o.connect(g); g.connect(ac.destination);
    o.start(); o.stop(ac.currentTime + d);
  } catch (e) {}
}
function toggleSound(b) { mute = !mute; b.textContent = "Sonido: " + (mute ? "no" : "sí"); }
function ranking(add) {
  try {
    let t = JSON.parse(localStorage.getItem("dep_top") || "[]");
    if (add) { t.push(add); t.sort((a, b) => b.p - a.p); t = t.slice(0, 5); localStorage.setItem("dep_top", JSON.stringify(t)); }
    return t;
  } catch (e) { return add ? [add] : []; }
}
function msg(t, c) { $("msg").className = c || ""; $("msg").textContent = t; }
function boom() {
  if (matchMedia("(prefers-reduced-motion:reduce)").matches) return;
  for (let i = 0; i < 26; i++) {
    const p = document.createElement("b");
    p.className = "pt";
    p.style.cssText = "left:" + (30 + Math.random() * 40) + "vw;background:hsl(" + Math.random() * 360 + ",90%,60%);--dx:" +
      ((Math.random() - .5) * 300) + "px;--dy:" + (-80 - Math.random() * 220) + "px";
    document.body.appendChild(p);
    setTimeout(() => p.remove(), 1100);
  }
}

/* FLUJO DEL JUEGO */
function startGame() {
  S = { li: 0, score: 0, lives: 5, streak: 0, name: $("nm").value.trim() || "Anónimo", phase: "predict" };
  $("start").hidden = $("endscr").hidden = true;
  $("game").hidden = false;
  loadCase();
}
function loadCase() {
  stopAuto();
  const l = L[S.li], v = {}, tr = [];
  l.run(v, n => tr.push([n, JSON.parse(JSON.stringify(v))]));
  Object.assign(S, {
    tr, st: -1, seen: false, phase: "predict", hints: 0, flawless: true,
    total: TIMES[floorOf()], left: TIMES[floorOf()], bad: -1, dead: [], bp: {}, pin: null, scan: false
  });
  S.opts = l.fix.map((o, k) => [o, k === 0]).sort(() => Math.random() - .5);
  S.pred = [[l.out, true], [l.exp, false], ["Error", false]].sort(() => Math.random() - .5);
  $("sl").max = tr.length - 1;
  msg("");
  clearInterval(timer);
  timer = setInterval(() => {
    if (S.phase === "play" || S.phase === "fix") {
      S.left = Math.max(0, S.left - 1);
      if (S.left === 0) msg("¡Tiempo agotado! Ya no hay bono de tiempo.", "err");
      head();
    }
  }, 1000);
  drawPred();
  draw();
}
function drawPred() {
  $("pred").innerHTML = "<h2>Antes de ejecutar: ¿qué imprimirá este programa?</h2>" +
    S.pred.map((o, k) => '<button class="opt" onclick="pickPred(' + k + ')">' + esc(o[0]) + "</button>").join("") +
    '<button onclick="endPred()">Saltar predicción</button>';
}
function pickPred(k) {
  if (S.pred[k][1]) { S.score += 20; snd(880, .15); msg("¡Buena predicción! +20", "good"); }
  else { snd(200, .15, "square"); msg("No era esa salida. Compruébalo ejecutando el programa.", "err"); }
  endPred(true);
}
function endPred(keep) { S.phase = "play"; if (!keep) msg(""); draw(); }
function head() {
  $("lvl").textContent = "Piso " + (floorOf() + 1) + " · Caso " + (S.li + 1) + "/" + L.length;
  $("hp").textContent = "♥".repeat(Math.max(0, S.lives));
  $("score").textContent = S.score;
  $("time").textContent = S.left;
}

/* DEPURACIÓN: pasos, auto, puntos de parada, escáner, vigilancia */
function go(n, fromAuto) {
  if (S.phase !== "play") return;
  if (!fromAuto) stopAuto();
  S.st = Math.max(-1, Math.min(S.tr.length - 1, n));
  if (S.st === S.tr.length - 1 && !S.seen) {
    S.seen = true; snd(220, .2, "square");
    msg("El programa terminó con un resultado incorrecto. Toca la línea responsable.", "err");
  } else snd(520, .05);
  draw();
}
function cont() {
  if (S.phase !== "play") return;
  let n = S.st + 1;
  while (n < S.tr.length - 1 && !S.bp[S.tr[n][0]]) n++;
  go(n);
}
function toggleAuto() {
  if (auto) { stopAuto(); return; }
  if (S.phase !== "play" || S.st >= S.tr.length - 1) return;
  $("bAuto").textContent = "⏸ Pausa";
  auto = setInterval(() => {
    const n = S.st + 1;
    go(n, true);
    if (n >= S.tr.length - 1 || S.bp[S.tr[n][0]]) stopAuto();
  }, +$("spd").value);
}
function stopAuto() { clearInterval(auto); auto = null; $("bAuto").textContent = "▶ Auto"; }
function toggleBp(i) { S.bp[i] = !S.bp[i]; snd(400, .04); draw(); }
function togglePin(k) { S.pin = S.pin === k ? null : k; draw(); }
function scan() {
  if (S.phase !== "play" || S.scan) return;
  S.scan = true; S.score = Math.max(0, S.score - 20); snd(300, .25, "triangle");
  msg("Escáner activo: las líneas apagadas son inocentes.", "good"); draw();
}
function hint() { S.hints++; msg("Pista: " + L[S.li].hint, "good"); snd(700, .1); }
function histText() {
  if (!S.pin) return "";
  const h = [];
  for (let i = 0; i <= S.st; i++) {
    const x = S.tr[i][1][S.pin];
    if (x !== undefined && (!h.length || fmt(x) !== h[h.length - 1])) h.push(fmt(x));
  }
  return "Historial de <b>" + S.pin + "</b>: " + (h.join(" → ") || "aún sin valor");
}

/* DIBUJO */
function draw() {
  const l = L[S.li], end = S.st === S.tr.length - 1, play = S.phase === "play";
  head();
  $("goal").innerHTML = "<h2>" + l.t + "</h2>" + l.goal;
  $("code").innerHTML = l.code.map((c, i) => {
    let k = "ln";
    if (S.phase === "fix" || S.phase === "done") { if (i === l.bug) k += " ok"; }
    else if (S.bad === i) k += " bad";
    else if (S.st >= 0 && S.tr[S.st][0] === i) k += " cur";
    if (S.scan && !l.sus.includes(i)) k += " dim";
    return '<div class="' + k + '" onclick="pick(' + i + ')"><i class="' + (S.bp[i] ? "on" : "") +
      '" title="Punto de parada" onclick="event.stopPropagation();toggleBp(' + i + ')">' + (i + 1) + "</i>" + esc(c) + "</div>";
  }).join("");
  const cur = S.st >= 0 ? S.tr[S.st][1] : {}, prev = S.st > 0 ? S.tr[S.st - 1][1] : {};
  $("vars").innerHTML = Object.keys(cur).map(k =>
    '<tr class="' + (fmt(cur[k]) !== fmt(prev[k]) ? "chg " : "") + (S.pin === k ? "pin" : "") +
    '" onclick="togglePin(\'' + k + '\')"><td>' + k + "</td><td>" + fmt(cur[k]) + "</td></tr>").join("")
    || '<tr><td colspan="2">Vacía. Da el primer paso.</td></tr>';
  $("hist").innerHTML = histText();
  $("out").innerHTML = end ? 'Salida: <span class="err">' + l.out + '</span> · Esperado: <span class="good">' + l.exp + "</span>" : "";
  $("sl").value = S.st;
  $("bBack").disabled = !play || S.st < 0;
  $("bStep").disabled = $("bCont").disabled = $("bAll").disabled = $("bAuto").disabled = !play || end;
  $("bScan").disabled = !play || S.scan;
  $("bHint").disabled = !play;
  $("pred").hidden = S.phase !== "predict";
  $("ctl").hidden = S.phase !== "play";
  $("fix").hidden = S.phase !== "fix";
  $("bNext").hidden = S.phase !== "done";
}

/* ACUSAR LA LÍNEA Y ELEGIR LA CORRECCIÓN */
function hurt() {
  S.lives--; S.flawless = false; snd(150, .3, "sawtooth");
  if (navigator.vibrate) navigator.vibrate(80);
  document.body.classList.remove("hit"); void document.body.offsetWidth; document.body.classList.add("hit");
  head();
  if (S.lives <= 0) { gameOver("Tu integridad llegó a cero."); return true; }
  return false;
}
function pick(i) {
  if (S.phase !== "play") return;
  if (!S.seen) { msg("Primero ejecuta el programa hasta el final para investigar."); return; }
  if (i === L[S.li].bug) {
    stopAuto(); S.phase = "fix"; S.bad = -1;
    S.score += Math.max(20, 60 - 15 * S.hints); snd(660, .15); snd(880, .2);
    msg("¡Línea encontrada! Ahora elige la corrección correcta.", "good");
    drawFix();
  } else {
    S.bad = i;
    if (hurt()) return;
    msg("Esa línea no causa el error. Observa dónde se desvían las variables.", "err");
  }
  draw();
}
function drawFix() {
  $("fix").innerHTML = "<h2>¿Cuál es la corrección?</h2>" + S.opts.map((o, k) =>
    '<button class="opt" ' + (S.dead[k] ? "disabled" : "") + ' onclick="chooseFix(' + k + ')">' + esc(o[0]) + "</button>").join("");
}
function chooseFix(k) {
  if (S.phase !== "fix") return;
  if (S.opts[k][1]) {
    S.phase = "done"; clearInterval(timer);
    const tb = Math.round(S.left / S.total * 30);
    S.streak = S.flawless ? S.streak + 1 : 0;
    const combo = S.flawless && S.streak > 1 ? (S.streak - 1) * 10 : 0;
    S.score += 40 + tb + combo; snd(784, .12); snd(988, .25); boom();
    msg("¡Corregido! +40 · tiempo +" + tb + (combo ? " · racha +" + combo : "") + ". " + L[S.li].why, "good");
    $("bNext").textContent = S.li === L.length - 1 ? "Ver resultado ▶" : "Siguiente caso ▶";
    draw(); $("bNext").focus();
  } else {
    S.dead[k] = true;
    if (hurt()) return;
    drawFix(); msg("Esa corrección no resuelve el bug. Pierdes una vida.", "err");
  }
}
function nextCase() { S.li++; if (S.li >= L.length) finish(); else loadCase(); }

/* FINAL */
function rank(p) {
  return p >= 1300 ? "Maestro Depurador" : p >= 950 ? "Depurador Senior" : p >= 550 ? "Depurador Junior" : "Aprendiz de Bugs";
}
function endScreen(title, sub) {
  clearInterval(timer); stopAuto(); S.phase = "end";
  $("game").hidden = true; $("endscr").hidden = false;
  const t = ranking({ n: S.name, p: S.score });
  $("endscr").innerHTML = "<h1>" + title + "</h1><p>" + sub + '</p><div class="big">' + S.score + " pts</div>" +
    "<p>Rango: <b>" + rank(S.score) + "</b></p><h2>Mejores puntajes</h2><table>" +
    t.map((r, i) => "<tr style='cursor:default'><td>" + (i + 1) + "</td><td>" + esc(r.n) + "</td><td>" + r.p + "</td></tr>").join("") +
    '</table><br><button class="main" onclick="startGame()">Jugar de nuevo</button>';
}
function finish() { snd(1047, .4); boom(); endScreen("¡Mazmorra depurada!", "Corregiste los " + L.length + " servidores con " + S.lives + " vidas restantes."); }
function gameOver(t) { endScreen("Servidor caído", t + " Llegaste al caso " + (S.li + 1) + " de " + L.length + "."); }

document.addEventListener("keydown", e => {
  if (S.phase !== "play" || $("game").hidden || ["INPUT", "SELECT"].includes(e.target.tagName)) return;
  if (e.key === "ArrowRight") go(S.st + 1);
  else if (e.key === "ArrowLeft") go(S.st - 1);
  else if (e.key.toLowerCase() === "c") cont();
  else if (e.key === " ") { e.preventDefault(); toggleAuto(); }
});
</script>
</body>
</html>

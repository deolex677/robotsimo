<!DOCTYPE html>
<html lang="pt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>RobotSim - Simulador Cartesiano</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #07101d;
    color: white;
}

header {
    padding: 18px;
    background: #101a2c;
    border-bottom: 1px solid #293a56;
}

h1 {
    margin: 0;
}

.sub {
    color: #9fb0ca;
    margin-top: 6px;
}

.app {
    display: grid;
    grid-template-columns: 300px 1fr;
    min-height: calc(100vh - 80px);
}

aside {
    padding: 18px;
    background: #111b2e;
    border-right: 1px solid #293a56;
}

section {
    margin-bottom: 20px;
}

input,
button {
    width: 100%;
    padding: 10px;
    margin-top: 7px;
    border-radius: 8px;
    border: 1px solid #344560;
    background: #17243a;
    color: white;
}

button {
    cursor: pointer;
    font-weight: bold;
}

button:hover {
    background: #263b5b;
}

.row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
}

label {
    display: block;
    margin-top: 8px;
    color: #aebed5;
}

.status {
    background: #0c1525;
    padding: 12px;
    border-radius: 10px;
    line-height: 1.7;
}

main {
    padding: 14px;
}

canvas {
    width: 100%;
    height: 70vh;
    background: #07101d;
    border: 1px solid #293a56;
    border-radius: 12px;
}

.info {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    margin-top: 10px;
}

.card {
    background: #111d31;
    border: 1px solid #293a56;
    padding: 10px 15px;
    border-radius: 8px;
}

.card b {
    display: block;
    color: #93a7c3;
    font-size: 12px;
}

.card span {
    font-size: 18px;
}

@media(max-width:800px) {

    .app {
        grid-template-columns: 1fr;
    }

    aside {
        border-right: none;
        border-bottom: 1px solid #293a56;
    }

    canvas {
        height: 55vh;
    }
}
</style>
</head>

<body>

<header>

<h1>🤖 RobotSim</h1>

<div class="sub">
Simulador de movimento de robôs em um plano cartesiano
</div>

</header>


<div class="app">

<aside>

<section>

<h3>🎮 Controlo</h3>

<button onclick="iniciar()">
▶ Iniciar
</button>

<button onclick="pausar()">
⏸ Pausar
</button>

<button onclick="reiniciar()">
↺ Reiniciar
</button>

</section>


<section>

<h3>📍 Destino</h3>

<div class="row">

<div>

<label>X</label>

<input
id="destinoX"
type="number"
value="5"
step="0.1">

</div>

<div>

<label>Y</label>

<input
id="destinoY"
type="number"
value="4"
step="0.1">

</div>

</div>

<button onclick="irParaDestino()">
🚀 Ir para destino
</button>

</section>


<section>

<h3>⚙️ Velocidade</h3>

<input
id="velocidade"
type="range"
min="0.2"
max="8"
value="2"
step="0.1">

</section>


<section>

<h3>📊 Estado do robô</h3>

<div class="status">

X:
<b id="posX">0.00</b>

<br>

Y:
<b id="posY">0.00</b>

<br>

Distância:
<b id="distancia">0.00</b>

<br>

Tempo:
<b id="tempo">0.0</b> s

<br>

Estado:
<b id="estado">Parado</b>

</div>

</section>


<section>

<h3>⌨️ Controlo manual</h3>

<p>
⬆️ Para cima<br>
⬇️ Para baixo<br>
⬅️ Para esquerda<br>
➡️ Para direita
</p>

</section>

</aside>


<main>

<canvas id="plano"></canvas>

<div class="info">

<div class="card">

<b>POSIÇÃO</b>

<span id="posicao">
(0.00, 0.00)
</span>

</div>


<div class="card">

<b>DESTINO</b>

<span id="destino">
(5.00, 4.00)
</span>

</div>


<div class="card">

<b>VELOCIDADE</b>

<span id="velocidadeTexto">
2.0
</span>

</div>


<div class="card">

<b>TRAJETÓRIA</b>

<span id="pontos">
1 pontos
</span>

</div>

</div>

</main>

</div>


<script>

const canvas =
document.getElementById("plano");

const ctx =
canvas.getContext("2d");


let largura;
let altura;

let robo = {

x: 0,
y: 0

};


let alvo = {

x: 5,
y: 4

};


let trajetoria = [

{x:0,y:0}

];


let executando = false;

let automatico = false;

let tempo = 0;

let ultimoTempo = 0;


const mundo = {

minX: -10,
maxX: 10,

minY: -8,
maxY: 8

};


function ajustarCanvas() {

const dpr =
window.devicePixelRatio || 1;

largura =
canvas.clientWidth;

altura =
canvas.clientHeight;

canvas.width =
largura * dpr;

canvas.height =
altura * dpr;

ctx.setTransform(
dpr,
0,
0,
dpr,
0,
0
);

desenhar();

}


window.addEventListener(
"resize",
ajustarCanvas
);


function telaX(x) {

return (

(x - mundo.minX) /
(mundo.maxX - mundo.minX)
*
largura

);

}


function telaY(y) {

return (

altura -

(y - mundo.minY) /
(mundo.maxY - mundo.minY)
*
altura

);

}


function desenhar() {

ctx.clearRect(
0,
0,
largura,
altura
);


// GRELHA

for(
let x = Math.ceil(mundo.minX);
x <= mundo.maxX;
x++
) {

ctx.beginPath();

ctx.strokeStyle =
x === 0
? "#9db1cc"
: "#1c3049";

ctx.moveTo(
telaX(x),
0
);

ctx.lineTo(
telaX(x),
altura
);

ctx.stroke();

}


for(
let y = Math.ceil(mundo.minY);
y <= mundo.maxY;
y++
) {

ctx.beginPath();

ctx.strokeStyle =
y === 0
? "#9db1cc"
: "#1c3049";

ctx.moveTo(
0,
telaY(y)
);

ctx.lineTo(
largura,
telaY(y)
);

ctx.stroke();

}


// EIXOS

ctx.fillStyle =
"#ffffff";

ctx.font =
"bold 14px Arial";

ctx.fillText(
"X",
largura - 20,
telaY(0) - 10
);

ctx.fillText(
"Y",
telaX(0) + 10,
20
);


// NÚMEROS

ctx.font =
"11px Arial";

ctx.fillStyle =
"#71859f";


for(
let x = -10;
x <= 10;
x++
) {

if(x !== 0) {

ctx.fillText(
x,
telaX(x)+3,
telaY(0)-4
);

}

}


for(
let y = -8;
y <= 8;
y++
) {

if(y !== 0) {

ctx.fillText(
y,
telaX(0)+5,
telaY(y)-4
);

}

}


// TRAJETÓRIA

if(
trajetoria.length > 1
) {

ctx.beginPath();

ctx.strokeStyle =
"#35d0ff";

ctx.lineWidth = 3;


trajetoria.forEach(
(p,i) => {

if(i === 0)

ctx.moveTo(
telaX(p.x),
telaY(p.y)
);

else

ctx.lineTo(
telaX(p.x),
telaY(p.y)
);

});


ctx.stroke();

}


// DESTINO

ctx.beginPath();

ctx.strokeStyle =
"#ffbd45";

ctx.lineWidth = 3;

ctx.arc(
telaX(alvo.x),
telaY(alvo.y),
10,
0,
Math.PI * 2
);

ctx.stroke();


// ROBÔ

const rx =
telaX(robo.x);

const ry =
telaY(robo.y);


ctx.beginPath();

ctx.fillStyle =
"#3b82f6";

ctx.arc(
rx,
ry,
16,
0,
Math.PI * 2
);

ctx.fill();


ctx.fillStyle =
"white";

ctx.fillRect(
rx-9,
ry-7,
18,
12
);


ctx.fillStyle =
"#07101d";

ctx.fillRect(
rx-6,
ry-4,
4,
4
);

ctx.fillRect(
rx+2,
ry-4,
4,
4
);


ctx.fillStyle =
"#3b82f6";

ctx.font =
"bold 12px Arial";

ctx.fillText(
"ROBÔ",
rx-18,
ry+30
);


atualizarInterface();

}


function atualizarInterface() {

const distancia =
Math.hypot(
alvo.x-robo.x,
alvo.y-robo.y
);


document.getElementById(
"posX"
).textContent =
robo.x.toFixed(2);


document.getElementById(
"posY"
).textContent =
robo.y.toFixed(2);


document.getElementById(
"distancia"
).textContent =
distancia.toFixed(2);


document.getElementById(
"tempo"
).textContent =
tempo.toFixed(1);


document.getElementById(
"posicao"
).textContent =
`(${robo.x.toFixed(2)}, ${robo.y.toFixed(2)})`;


document.getElementById(
"destino"
).textContent =
`(${alvo.x.toFixed(2)}, ${alvo.y.toFixed(2)})`;


document.getElementById(
"velocidadeTexto"
).textContent =
Number(
document.getElementById(
"velocidade"
).value
).toFixed(1);


document.getElementById(
"pontos"
).textContent =
trajetoria.length +
" pontos";


document.getElementById(
"estado"
).textContent =
executando
?
(automatico
? "Movendo para destino"
: "Controlo manual")
:
"Parado";

}


function adicionarPonto() {

const ultimo =
trajetoria[
trajetoria.length-1
];


if(
!ultimo ||
Math.hypot(
robo.x-ultimo.x,
robo.y-ultimo.y
) > 0.015
) {

trajetoria.push({

x: robo.x,
y: robo.y

});

}

}


function irParaDestino() {

alvo.x =
Number(
document.getElementById(
"destinoX"
).value
) || 0;


alvo.y =
Number(
document.getElementById(
"destinoY"
).value
) || 0;


automatico = true;

executando = true;

tempo = 0;

ultimoTempo =
performance.now();

requestAnimationFrame(
loop
);

}


function iniciar() {

executando = true;

automatico = false;

ultimoTempo =
performance.now();

requestAnimationFrame(
loop
);

}


function pausar() {

executando = false;

automatico = false;

desenhar();

}


function reiniciar() {

executando = false;

automatico = false;

robo = {

x:0,
y:0

};

trajetoria = [

{x:0,y:0}

];

tempo = 0;

desenhar();

}


function loop(agora) {

if(!executando)

return;


const dt =
Math.min(
(agora-ultimoTempo)/1000,
0.05
);


ultimoTempo =
agora;

tempo += dt;


const velocidade =
Number(
document.getElementById(
"velocidade"
).value
);


if(automatico) {

const dx =
alvo.x-robo.x;

const dy =
alvo.y-robo.y;

const distancia =
Math.hypot(dx,dy);


if(distancia < 0.03) {

robo.x =
alvo.x;

robo.y =
alvo.y;

executando = false;

automatico = false;

}


else {

const passo =
Math.min(
velocidade*dt,
distancia
);


robo.x +=
dx/distancia*
passo;


robo.y +=
dy/distancia*
passo;


adicionarPonto();

}

}


desenhar();


if(executando)

requestAnimationFrame(
loop
);

}


document.addEventListener(
"keydown",
function(e) {

if(
[
"ArrowUp",
"ArrowDown",
"ArrowLeft",
"ArrowRight"
].includes(e.key)
) {

e.preventDefault();

executando = false;

automatico = false;


const passo =
Number(
document.getElementById(
"velocidade"
).value
) * 0.15;


if(e.key === "ArrowUp")

robo.y += passo;


if(e.key === "ArrowDown")

robo.y -= passo;


if(e.key === "ArrowLeft")

robo.x -= passo;


if(e.key === "ArrowRight")

robo.x += passo;


robo.x =
Math.max(
mundo.minX,
Math.min(
mundo.maxX,
robo.x
)
);


robo.y =
Math.max(
mundo.minY,
Math.min(
mundo.maxY,
robo.y
)
);


adicionarPonto();

desenhar();

}

});


document.getElementById(
"velocidade"
).addEventListener(
"input",
desenhar
);


ajustarCanvas();

</script>

</body>
</html># robotsimo

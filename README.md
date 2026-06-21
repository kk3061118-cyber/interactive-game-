# interactive-game-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Zombie Survival</title>
 
   //kk3061118@gmail.com
<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
overflow:hidden;
background:#0f172a;
}

canvas{
display:block;
background:#111827;
}

#hud{
position:absolute;
top:10px;
left:10px;
color:white;
background:rgba(0,0,0,.5);
padding:12px;
border-radius:10px;
font-size:18px;
}

.healthBar{
width:220px;
height:20px;
background:#333;
border:2px solid white;
margin-top:5px;
}

#healthFill{
height:100%;
width:100%;
background:lime;
}

#waveText{
position:absolute;
top:50%;
left:50%;
transform:translate(-50%,-50%);
font-size:60px;
font-weight:bold;
color:yellow;
display:none;
}
</style>
</head>
<body>

<div id="hud">

❤️ Health

<div class="healthBar">
<div id="healthFill"></div>
</div>

<br>

⭐ Score:
<span id="score">0</span>

<br><br>

🌊 Wave:
<span id="wave">1</span>

<br><br>

💰 Coins:
<span id="coins">0</span>

</div>

<div id="waveText"></div>

<canvas id="game"></canvas>

<script>

const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

const scoreEl=document.getElementById("score");
const waveEl=document.getElementById("wave");
const coinsEl=document.getElementById("coins");
const healthFill=document.getElementById("healthFill");
const waveText=document.getElementById("waveText");

let score=0;
let coins=0;
let wave=1;
let health=100;

const player={
x:canvas.width/2,
y:canvas.height/2,
r:20,
speed:5
};

const keys={};
const bullets=[];
const zombies=[];
const particles=[];

let mouse={
x:0,
y:0
};

document.addEventListener("keydown",e=>{
keys[e.key.toLowerCase()]=true;
});

document.addEventListener("keyup",e=>{
keys[e.key.toLowerCase()]=false;
});

canvas.addEventListener("mousemove",e=>{
mouse.x=e.clientX;
mouse.y=e.clientY;
});

canvas.addEventListener("click",shoot);

function shoot(){

let angle=Math.atan2(
mouse.y-player.y,
mouse.x-player.x
);

bullets.push({
x:player.x,
y:player.y,
dx:Math.cos(angle)*10,
dy:Math.sin(angle)*10
});

}

function spawnZombie(){

let side=Math.floor(Math.random()*4);

let x,y;

if(side===0){
x=Math.random()*canvas.width;
y=-30;
}
else if(side===1){
x=canvas.width+30;
y=Math.random()*canvas.height;
}
else if(side===2){
x=Math.random()*canvas.width;
y=canvas.height+30;
}
else{
x=-30;
y=Math.random()*canvas.height;
}

zombies.push({
x,
y,
r:18,
speed:1+wave*0.1
});

}

setInterval(()=>{

for(let i=0;i<wave;i++){
spawnZombie();
}

},2500);

setInterval(()=>{

wave++;

waveEl.textContent=wave;

waveText.style.display="block";
waveText.innerHTML="WAVE "+wave;

setTimeout(()=>{
waveText.style.display="none";
},1500);

},15000);

function update(){

if(keys["w"]) player.y-=player.speed;
if(keys["s"]) player.y+=player.speed;
if(keys["a"]) player.x-=player.speed;
if(keys["d"]) player.x+=player.speed;

player.x=Math.max(0,
Math.min(canvas.width,player.x));

player.y=Math.max(0,
Math.min(canvas.height,player.y));

for(let i=bullets.length-1;i>=0;i--){

let b=bullets[i];

b.x+=b.dx;
b.y+=b.dy;

if(
b.x<0||
b.y<0||
b.x>canvas.width||
b.y>canvas.height
){
bullets.splice(i,1);
}
}

for(let i=zombies.length-1;i>=0;i--){

let z=zombies[i];

let angle=Math.atan2(
player.y-z.y,
player.x-z.x
);

z.x+=Math.cos(angle)*z.speed;
z.y+=Math.sin(angle)*z.speed;

if(
Math.hypot(
player.x-z.x,
player.y-z.y
)<30
){

health-=0.15;

if(health<=0){

alert(
"💀 GAME OVER\nScore: "+score
);

location.reload();

}

}

for(let j=bullets.length-1;j>=0;j--){

let b=bullets[j];

if(
Math.hypot(
b.x-z.x,
b.y-z.y
)<20
){

for(let k=0;k<8;k++){

particles.push({
x:z.x,
y:z.y,
vx:(Math.random()-.5)*5,
vy:(Math.random()-.5)*5,
life:25
});

}

zombies.splice(i,1);
bullets.splice(j,1);

score+=10;
coins+=5;

break;
}
}
}

for(let i=particles.length-1;i>=0;i--){

particles[i].x+=particles[i].vx;
particles[i].y+=particles[i].vy;
particles[i].life--;

if(particles[i].life<=0){
particles.splice(i,1);
}
}

scoreEl.textContent=score;
coinsEl.textContent=coins;

healthFill.style.width=
health+"%";

}

function draw(){

ctx.clearRect(
0,0,
canvas.width,
canvas.height
);

for(let i=0;i<120;i++){

ctx.fillStyle="white";

ctx.fillRect(
(i*97)%canvas.width,
(i*53)%canvas.height,
2,
2
);

}

ctx.fillStyle="#00ff88";

ctx.beginPath();
ctx.arc(
player.x,
player.y,
player.r,
0,
Math.PI*2
);
ctx.fill();

ctx.strokeStyle="white";

ctx.beginPath();
ctx.arc(
mouse.x,
mouse.y,
12,
0,
Math.PI*2
);
ctx.stroke();

ctx.fillStyle="yellow";

bullets.forEach(b=>{

ctx.beginPath();

ctx.arc(
b.x,
b.y,
4,
0,
Math.PI*2
);

ctx.fill();

});

zombies.forEach(z=>{

ctx.fillStyle="crimson";

ctx.beginPath();

ctx.arc(
z.x,
z.y,
z.r,
0,
Math.PI*2
);

ctx.fill();

});

ctx.fillStyle="orange";

particles.forEach(p=>{

ctx.fillRect(
p.x,
p.y,
3,
3
);

});

}

function loop(){

update();
draw();

requestAnimationFrame(loop);

}

loop();

</script>

</body>
</html>
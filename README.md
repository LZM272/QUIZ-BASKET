# QUIZ-BASKET
🏀 Quiz Basket Ultimate est un jeu de quiz interactif avec badges, classement et questions infinies. Teste ta culture basket (NBA, règles, joueurs), débloque des récompenses et vise le rang GOAT !
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Quiz Basket Ultimate 🏀</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap" rel="stylesheet">
<style>
body{
  margin:0;
  font-family:'Bebas Neue',Arial,sans-serif;
  background:radial-gradient(circle at top,#1c2541,#000);
  color:white;
  text-align:center;
}
.container{
  max-width:800px;
  margin:auto;
  padding:20px;
}
h1{font-size:48px; letter-spacing:2px;}
.box{
  background:#0b132b;
  padding:25px;
  border-radius:20px;
  box-shadow:0 0 30px rgba(0,0,0,0.7);
}
button,select,input{
  width:100%;
  padding:14px;
  margin:10px 0;
  border:none;
  border-radius:12px;
  font-size:18px;
}
button{
  background:linear-gradient(135deg,#ff6a00,#ffd500);
  cursor:pointer;
  box-shadow:0 0 15px rgba(255,140,0,0.6);
}
button:hover{transform:scale(1.04)}
.timer{color:#4cc9f0;font-size:22px}
.badge{
  display:inline-block;
  background:gold;
  color:black;
  padding:8px 14px;
  border-radius:20px;
  margin:5px;
}
.leaderboard{margin-top:20px;text-align:left}
.leaderboard div{padding:6px;border-bottom:1px solid #333}
</style>
</head>
<body>

<div class="container">
<h1>🏀 QUIZ BASKET ULTIMATE</h1>

<div class="box" id="app">
<input id="pseudo" placeholder="Ton pseudo">
<button onclick="savePlayer()">💾 Sauvegarder</button>
<p id="playerInfo"></p>

<select id="category">
  <option value="nba">NBA</option>
  <option value="regles">Règles</option>
  <option value="joueurs">Joueurs</option>
  <option value="histoire">Histoire</option>
</select>

<select id="difficulty">
  <option value="easy">Facile</option>
  <option value="medium">Moyen</option>
  <option value="hard">Difficile</option>
  <option value="legend">Légende</option>
</select>

<button onclick="startQuiz()">▶️ DÉMARRER LE QUIZ</button>

<hr>
<div class="timer" id="timer"></div>
<h2 id="question"></h2>
<div id="answers"></div>
<p id="score"></p>
<div id="badges"></div>
<div class="leaderboard" id="leaderboard"></div>
</div>
</div>

<audio id="swish" src="https://actions.google.com/sounds/v1/sports/basketball_bounce.ogg"></audio>
<audio id="buzzer" src="https://actions.google.com/sounds/v1/alarms/buzzer.ogg"></audio>

<script type="module">
// ===== FIREBASE =====
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, query, orderBy, limit } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "TA_API_KEY",
  authDomain: "TON_PROJET.firebaseapp.com",
  projectId: "TON_PROJET",
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// ===== JOUEUR =====
let player = localStorage.getItem('player') || '';
if(player) document.getElementById('playerInfo').textContent = '👤 ' + player;
window.savePlayer = () => {
  player = document.getElementById('pseudo').value;
  localStorage.setItem('player', player);
  document.getElementById('playerInfo').textContent = '👤 ' + player;
}

// ===== QUESTIONS (BASE POUR 500) =====
const baseQuestions = [
 {q:"Combien de joueurs sur le terrain par équipe ?",a:["4","5","6","7"],c:1},
 {q:"Combien vaut un lancer franc ?",a:["1","2","3","4"],c:0},
 {q:"Quel joueur a marqué 100 points en un match ?",a:["Jordan","Kobe","Wilt Chamberlain","LeBron"],c:2},
 {q:"Durée d'un quart-temps NBA ?",a:["10 min","12 min","15 min","8 min"],c:1},
 {q:"Quelle équipe a le plus de titres NBA ?",a:["Lakers","Bulls","Celtics","Warriors"],c:2}
];

let questions=[],index=0,score=0,time=20,timer,badges=[];

window.startQuiz = () => {
  questions=[];
  for(let i=0;i<500;i++) questions.push(baseQuestions[i%baseQuestions.length]);
  questions.sort(()=>Math.random()-0.5);
  index=0;score=0;badges=[];
  showQuestion();
}

function showQuestion(){
  if(index>=questions.length){endQuiz();return;}
  clearInterval(timer);
  time=20;
  document.getElementById('timer').textContent='⏱️ '+time;
  const q=questions[index];
  document.getElementById('question').textContent=q.q;
  const a=document.getElementById('answers');a.innerHTML='';
  q.a.forEach((txt,i)=>{
    const b=document.createElement('button');
    b.textContent=txt;
    b.onclick=()=>answer(i);
    a.appendChild(b);
  });
  document.getElementById('score').textContent='Score : '+score;
  timer=setInterval(()=>{
    time--;
    document.getElementById('timer').textContent='⏱️ '+time;
    if(time===0){index++;showQuestion();}
  },1000);
}

function answer(i){
  clearInterval(timer);
  if(i===questions[index].c){score++;document.getElementById('swish').play();}
  index++;showQuestion();
}

async function endQuiz(){
  document.getElementById('buzzer').play();
  unlockBadges();
  await addDoc(collection(db,"scores"),{player,score,date:Date.now()});
  const lb = await getLeaderboard();
  document.getElementById('app').innerHTML=`<h2>🏁 FIN DU QUIZ</h2><p>${player} : ${score}</p>${badges.map(b=>'<span class=badge>'+b+'</span>').join('')}<h3>🌍 Classement mondial</h3>${lb}`;
}

function unlockBadges(){
  if(score>=50)badges.push('🏅 Rookie');
  if(score>=150)badges.push('🥈 All-Star');
  if(score>=300)badges.push('🥇 MVP');
  if(score===500)badges.push('👑 GOAT');
}

async function getLeaderboard(){
  const q = query(collection(db,"scores"),orderBy("score","desc"),limit(10));
  const snap = await getDocs(q);
  let html='';
  snap.forEach((d,i)=>{const s=d.data();html+=`<div>${i+1}. ${s.player} — ${s.score}</div>`});
  return html;
}
</script>
</body>
</html>

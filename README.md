# Recreate the updated HTML game file since previous execution failed
html_file_content = """<!DOCTYPE html>
<html lang='en'>
<head>
<meta charset='utf-8' />
<meta name='viewport' content='width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no' />
<title>Cellphone Vocab Game</title>
<style>
:root {--bg:#0f172a;--card:#111827;--text:#e5e7eb;--muted:#94a3b8;--accent:#22c55e;--wrong:#ef4444;}
body {margin:0;background:linear-gradient(180deg,#0b1026,#0f172a 40%,#0b1026);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Noto Sans,Apple SD Gothic Neo,sans-serif;min-height:100svh;display:grid;place-items:center;padding:16px;}
.app{width:min(720px,100%);} .card{background:rgba(17,24,39,.9);border:1px solid #1f2937;border-radius:18px;padding:16px;box-shadow:0 8px 28px rgba(0,0,0,.35);}
header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:10px;}
.title{font-weight:800;font-size:clamp(18px,2.6vw,20px);}
.pill{display:inline-flex;align-items:center;gap:8px;padding:6px 10px;border-radius:999px;background:#111827;border:1px solid #334155;color:var(--muted);font-size:12px;}
.barWrap{height:8px;background:#0b1220;border-radius:999px;border:1px solid #1f2937;overflow:hidden;}
.bar{height:100%;width:0%;background:linear-gradient(90deg,#22c55e,#a3e635);transition:width .3s ease;}
.panel{display:grid;gap:12px;margin:10px 0 16px;}
.prompt{font-size:clamp(18px,3.8vw,22px);line-height:1.35;background:#0b1220;border:1px solid #1f2937;padding:14px;border-radius:14px;}
.prompt small{display:block;color:var(--muted);font-size:12px;margin-top:6px;}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
.btn{border:1px solid #334155;background:linear-gradient(180deg,#111827,#0b1220);color:var(--text);padding:14px;border-radius:14px;font-weight:700;letter-spacing:.3px;display:flex;align-items:center;justify-content:center;min-height:56px;transition:transform .06s ease,border-color .2s ease,box-shadow .2s ease;}
.btn:active{transform:scale(.98);} .btn.correct{border-color:#22c55e;box-shadow:0 0 0 2px rgba(34,197,94,.25) inset;} .btn.wrong{border-color:var(--wrong);box-shadow:0 0 0 2px rgba(239,68,68,.25) inset;}
.hud{display:flex;align-items:center;justify-content:space-between;gap:10px;color:var(--muted);font-size:12px;} .big{font-size:32px;font-weight:800;}
.footer{display:flex;gap:8px;justify-content:space-between;margin-top:6px;} .smallBtn{flex:1;border:1px solid #334155;background:#0b1220;color:var(--text);padding:10px;border-radius:12px;font-weight:700;}
.center{text-align:center;} .hidden{display:none!important;}
</style>
</head>
<body>
<div class='app'>
<div class='card'>
<header><div class='title'>📱 Cellphone Vocab Game</div><div class='pill' id='scorePill'>Score: <strong id='score'>0</strong></div></header>
<div class='panel'><div class='barWrap'><div class='bar' id='progress'></div></div><div class='hud'><span id='qCount'>Question 1 / 10</span><span id='streak' class='pill'>Streak: 0</span></div><div class='prompt' id='prompt'>—</div><div class='grid' id='choices'></div></div>
<div class='footer'><button class='smallBtn' id='modeBtn'>🔁 Mode: Definition → Word</button><button class='smallBtn' id='restartBtn'>🔄 Restart</button></div>
</div>
<div id='end' class='card hidden' style='margin-top:12px;'><div class='center'><div class='big' id='finalScore'>0 / 10</div><p id='feedback' style='color:var(--muted);margin-top:4px;'>Nice work!</p><div style='display:flex;gap:8px;margin-top:10px;'><button class='smallBtn' onclick='startGame()'>Play Again</button><button class='smallBtn' onclick='toggleMode()'>Switch Mode</button></div></div></div>
</div>
<script>
const BANK=[{w:'avoid',d:'to stay away from something'},{w:'vanish',d:'to disappear suddenly'},{w:'crash',d:'to hit something hard and make a loud noise'},{w:'remain',d:'to stay in the same place or condition'},{w:'realize',d:'to suddenly understand something'},{w:'notice',d:'to see or hear something'},{w:'vibrate',d:'to move back and forth very fast'},{w:'spend',d:'to use time or money'}];
const promptEl=document.getElementById('prompt'),choicesEl=document.getElementById('choices'),progressEl=document.getElementById('progress'),scoreEl=document.getElementById('score'),qCountEl=document.getElementById('qCount'),streakEl=document.getElementById('streak'),endCard=document.getElementById('end'),finalScoreEl=document.getElementById('finalScore'),feedbackEl=document.getElementById('feedback'),modeBtn=document.getElementById('modeBtn'),restartBtn=document.getElementById('restartBtn');
let MODE=0,q=0,total=10,score=0,streak=0;
function shuffle(a){for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];}return a;}
function pickQuestion(){const pool=shuffle([...BANK]);const correct=pool[0];const distractors=pool.slice(1).filter(x=>x.w!==correct.w).slice(0,3);return{correct,options:shuffle([correct,...distractors])};}
function setProgress(){progressEl.style.width=((q/total)*100)+'%';qCountEl.textContent=`Question ${q+1} / ${total}`;scoreEl.textContent=score;streakEl.textContent=`Streak: ${streak}`;}
function renderQuestion(){const{correct,options}=pickQuestion();const tip=MODE===0?'Choose the correct WORD.':'Choose the correct DEFINITION.';const emoji=MODE===0?'🧩':'📖';promptEl.innerHTML=`${emoji} <strong>${MODE===0?correct.d:correct.w}</strong><small>${tip}</small>`;choicesEl.innerHTML='';options.forEach(opt=>{const btn=document.createElement('button');btn.className='btn';btn.textContent=MODE===0?opt.w:opt.d;btn.onclick=()=>handleAnswer(opt.w===correct.w,btn);choicesEl.appendChild(btn);});}
function handleAnswer(isCorrect,btn){if(isCorrect){btn.classList.add('correct');score++;streak++;}else{btn.classList.add('wrong');streak=0;}[...choicesEl.children].forEach(b=>b.disabled=true);setTimeout(()=>{q++;setProgress();if(q>=total)endGame();else renderQuestion();},700);}
function endGame(){finalScoreEl.textContent=`${score} / ${total}`;let fb='';if(score===total)fb='🎉 Congratulations! 10/10!! You are a vocabulary master!';else if(score>=8)fb='👏 Good job! You got most of them right. Want to try one more time?';else if(score>=5)fb='👍 Not bad! You’re learning fast—give it another try!';else fb='💪 Keep practicing! You’ll get better every time!';feedbackEl.textContent=fb;endCard.classList.remove('hidden');}
function resetState(){q=0;score=0;streak=0;setProgress();}function startGame(){endCard.classList.add('hidden');resetState();renderQuestion();}function toggleMode(){MODE=MODE===0?1:0;modeBtn.textContent=MODE===0?'🔁 Mode: Definition → Word':'🔁 Mode: Word → Definition';startGame();}modeBtn.addEventListener('click',toggleMode);restartBtn.addEventListener('click',startGame);startGame();
</script>
</body></html>"""
path="/mnt/data/cellphone_vocab_game_final.html"
with open(path,"w",encoding="utf-8") as f:
    f.write(html_file_content)
path

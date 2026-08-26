html = r'''<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>ROLL A VMA</title>
<style>
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;background:#050509;color:#fff;font-family:Arial,sans-serif}
body{overflow-x:hidden}
button{font:inherit}
#app{min-height:100vh;padding:24px}
.top{display:flex;justify-content:space-between;gap:20px;align-items:center;flex-wrap:wrap}
h1{margin:0;font-size:clamp(28px,5vw,52px);letter-spacing:4px}
.stats{display:flex;gap:10px;flex-wrap:wrap}
.stat{padding:10px 14px;border:1px solid #444;border-radius:10px;background:#111}
.controls{display:flex;gap:10px;flex-wrap:wrap;margin:24px 0}
button{border:1px solid #666;background:#15151c;color:white;padding:12px 16px;border-radius:10px;cursor:pointer}
button:hover{filter:brightness(1.3)}
.roll{font-size:22px;font-weight:bold;padding:16px 28px}
#result{min-height:110px;display:flex;align-items:center;justify-content:center;text-align:center;border:1px solid #333;border-radius:16px;background:#0b0b11;font-size:30px;font-weight:bold;padding:20px}
#sub{margin-top:8px;text-align:center;color:#aaa}
.panel{margin-top:22px;border:1px solid #292932;border-radius:16px;background:#0b0b10;padding:18px}
.panel h2{margin-top:0}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:10px}
.card{padding:14px;border:1px solid #333;border-radius:12px;background:#111118}
.card.locked{opacity:.45}
.card .name{font-weight:bold;font-size:18px}
.card .rarity{font-size:12px;color:#aaa;margin-top:5px}
.card button{margin-top:10px;width:100%;padding:8px}
.on{border-color:#fff}
.off{border-color:#555;color:#888}

#cutscene{position:fixed;inset:0;background:#020204;display:none;z-index:1000;overflow:hidden}
#cutscene.active{display:block}
#sceneBg{position:absolute;inset:0;overflow:hidden}
#sceneArt{position:absolute;inset:0}
#sceneText{position:absolute;z-index:50;inset:0;display:flex;align-items:center;justify-content:center;text-align:center;padding:8vw;font-size:clamp(28px,6vw,76px);font-weight:900;letter-spacing:4px;text-shadow:0 0 12px #fff,0 0 35px currentColor}
#sceneSmall{position:absolute;z-index:60;left:50%;bottom:12%;transform:translateX(-50%);font-size:16px;letter-spacing:3px;text-align:center;color:#ddd}
#skip{position:absolute;z-index:100;right:20px;top:20px}
.layer{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);pointer-events:none}
.symbol{position:absolute;line-height:1;font-weight:900;user-select:none;animation:float var(--d) linear infinite}
.shape{position:absolute;border:2px solid currentColor;animation:spin var(--d) linear infinite}
.line{position:absolute;height:2px;transform-origin:left center;animation:lineMove var(--d) ease-in-out infinite}
.word{position:absolute;font-weight:900;white-space:nowrap;animation:wordMove var(--d) ease-in-out infinite}
.noise{position:absolute;inset:-20%;background:repeating-linear-gradient(0deg,transparent 0 3px,rgba(255,255,255,.035) 4px 5px);mix-blend-mode:screen}
@keyframes float{0%{transform:translate(0,0) rotate(0) scale(.6);opacity:0}15%{opacity:1}50%{transform:translate(var(--x),var(--y)) rotate(var(--r)) scale(1)}100%{transform:translate(calc(var(--x)*-1),calc(var(--y)*-1)) rotate(calc(var(--r)*-1)) scale(1.5);opacity:0}}
@keyframes spin{0%{transform:translate(-50%,-50%) rotate(0) scale(.2);opacity:0}20%{opacity:1}100%{transform:translate(-50%,-50%) rotate(360deg) scale(2.5);opacity:0}}
@keyframes lineMove{0%,100%{transform:rotate(var(--a)) scaleX(.2);opacity:.1}50%{transform:rotate(calc(var(--a) + 100deg)) scaleX(1.8);opacity:1}}
@keyframes wordMove{0%{transform:translate(-20vw,10vh) rotate(-12deg);opacity:0}25%{opacity:1}100%{transform:translate(20vw,-10vh) rotate(12deg);opacity:0}}
@keyframes jitter{0%,100%{transform:translate(0)}20%{transform:translate(8px,-5px)}40%{transform:translate(-7px,6px)}60%{transform:translate(5px,8px)}80%{transform:translate(-9px,-4px)}}
.jitter{animation:jitter .07s infinite}
@keyframes flash{0%{opacity:0}20%{opacity:1}100%{opacity:0}}
.flash{position:absolute;inset:0;background:#fff;z-index:80;animation:flash .22s forwards}
.fade{animation:fade .7s forwards}
@keyframes fade{from{opacity:0}to{opacity:1}}
@keyframes zoom{from{transform:scale(.1) rotate(-20deg);opacity:0}to{transform:scale(1) rotate(0);opacity:1}}
.zoom{animation:zoom .7s cubic-bezier(.2,1.5,.3,1) forwards}
</style>
</head>
<body>
<div id="app">
  <div class="top">
    <h1>ROLL A VMA</h1>
    <div class="stats">
      <div class="stat">ROLLS: <b id="rolls">0</b></div>
      <div class="stat">LUCK: <b id="luck">1x</b></div>
      <div class="stat">BREAKTHROUGHS: <b id="breaks">0</b></div>
    </div>
  </div>

  <div class="controls">
    <button class="roll" onclick="roll()">ROLL</button>
    <button onclick="roll10()">ROLL ×10</button>
    <button onclick="resetSave()">RESET SAVE</button>
  </div>

  <div id="result">READY.</div>
  <div id="sub">Your collection and cutscene settings save automatically.</div>

  <div class="panel">
    <h2>CHARACTER INDEX</h2>
    <div id="index" class="grid"></div>
  </div>
</div>

<div id="cutscene">
  <div id="sceneBg"></div>
  <div id="sceneArt"></div>
  <div id="sceneText"></div>
  <div id="sceneSmall"></div>
  <button id="skip" onclick="finishCutscene()">SKIP CUTSCENE</button>
</div>

<script>
/*
  ROLL A VMA
  Save data: localStorage
  Cutscene rule:
  - Legendary and above receive cutscenes.
  - Every person/band has a different visual language.
  - No expanding rings, shockwaves, sideways comets, or generic repeated rays.
  - Items/simple characters have no cutscene.
  - Breakthroughs occur at 100,200,300... up to 99,999,999 rolls.
*/

const CHARACTERS = [
 {name:"Tristan",rarity:"Legendary",type:"person",theme:"drums",color:"#e7e7e7"},
 {name:"Cean",rarity:"Epic",type:"person",theme:"bass",color:"#8e7dff"},
 {name:"Nather",rarity:"Legendary",type:"person",theme:"rhythm",color:"#ff9a3c"},
 {name:"Jazlin",rarity:"Divine",type:"person",theme:"voice",color:"#ff5bc8"},
 {name:"Liam",rarity:"Celestial",type:"person",theme:"guitar",color:"#55d8ff"},
 {name:"Leo",rarity:"Secret",type:"person",theme:"legacy",color:"#9fd8ff"},
 {name:"Eduard",rarity:"Secret",type:"person",theme:"vocalMemory",color:"#e0b86b"},
 {name:"Tcell",rarity:"Secret",type:"band",theme:"tcell",color:"#8aff9a"},
 {name:"Lookphoria",rarity:"Ultimate",type:"band",theme:"lookphoria",color:"#d8b56a"},
 {name:"VMA",rarity:"???",type:"ultimate",theme:"vma",color:"#ffffff"},

 {name:"Drumpad",rarity:"Common",type:"item",theme:"none"},
 {name:"Guitar Habit Amplifier",rarity:"Rare",type:"item",theme:"none"},
 {name:"Tristan's Drumsticks",rarity:"Rare",type:"item",theme:"none"},
 {name:"Liam's Snapped String",rarity:"Rare",type:"item",theme:"none"},
 {name:"My Snapped String",rarity:"Rare",type:"item",theme:"none"}
];

const rarityWeight = {
 Common:650, Rare:220, Epic:80, Legendary:35, Divine:12,
 Celestial:5, Secret:1.5, Ultimate:.25, "???":.02
};

let state = JSON.parse(localStorage.getItem("rollAVMAState") || "null") || {
 rolls:0, breakthroughs:0, collected:{}, cutscene:{}
};

function save(){localStorage.setItem("rollAVMAState",JSON.stringify(state));}
function $(id){return document.getElementById(id)}
function updateUI(){
 $("rolls").textContent=state.rolls.toLocaleString();
 $("breaks").textContent=Math.floor(state.rolls/100).toLocaleString();
 $("luck").textContent=(state.rolls>0 && state.rolls%100===0 ? "10x" : "1x");
 renderIndex(); save();
}
function renderIndex(){
 $("index").innerHTML="";
 CHARACTERS.forEach(c=>{
   const card=document.createElement("div");
   card.className="card"+(state.collected[c.name]?"":" locked");
   const status=state.collected[c.name]?"COLLECTED":"???";
   card.innerHTML=`<div class="name">${status} ${c.name}</div>
   <div class="rarity">${c.rarity}</div>`;
   if(state.collected[c.name]){
     const b=document.createElement("button");
     const enabled=state.cutscene[c.name]!==false;
     b.className=enabled?"on":"off";
     b.textContent=enabled?"CUTSCENE: ON":"CUTSCENE: OFF";
     if(c.type==="item") {b.disabled=true;b.textContent="NO CUTSCENE";}
     else b.onclick=()=>{state.cutscene[c.name]=!enabled;updateUI()};
     card.appendChild(b);
   }
   $("index").appendChild(card);
 });
}

function weightedPick(){
 let pool=[];
 CHARACTERS.forEach(c=>{
   for(let i=0;i<rarityWeight[c.rarity]*10;i++) pool.push(c);
 });
 return pool[Math.floor(Math.random()*pool.length)];
}

function roll10(){
 for(let i=0;i<10;i++) roll(true);
 if(!document.getElementById("cutscene").classList.contains("active")) updateUI();
}

function roll(fromMulti=false){
 state.rolls++;
 let item=weightedPick();

 // Guaranteed Liam after 30 rolls if not yet collected.
 if(state.rolls>=30 && !state.collected.Liam && state.rolls%30===0) item=CHARACTERS.find(x=>x.name==="Liam");

 state.collected[item.name]=true;

 const breakthrough = state.rolls%100===0;
 $("result").innerHTML=`<b>${item.name}</b><br><small>${item.rarity}</small>`;
 $("sub").textContent=breakthrough
   ? `BREAKTHROUGH ${Math.floor(state.rolls/100)}`
   : `${state.rolls.toLocaleString()} rolls`;

 updateUI();

 if(breakthrough){
   playBreakthrough(Math.floor(state.rolls/100));
 } else if(item.type!=="item" && (item.rarity==="Legendary"||item.rarity==="Divine"||item.rarity==="Celestial"||item.rarity==="Secret"||item.rarity==="Ultimate"||item.rarity==="???") && state.cutscene[item.name]!==false){
   playCharacter(item);
 }
}

function audioBeep(freq,dur=0.18,type="sine",gain=.035){
 try{
  const A=window.AudioContext||window.webkitAudioContext;
  if(!A)return;
  const ctx=audioBeep.ctx||(audioBeep.ctx=new A());
  const o=ctx.createOscillator(),g=ctx.createGain();
  o.type=type;o.frequency.value=freq;g.gain.value=gain;
  o.connect(g);g.connect(ctx.destination);
  o.start();g.gain.exponentialRampToValueAtTime(.0001,ctx.currentTime+dur);
  o.stop(ctx.currentTime+dur);
 }catch(e){}
}

let sceneTimer=[];
function clearScene(){
 sceneTimer.forEach(clearTimeout);sceneTimer=[];
 $("sceneBg").innerHTML="";$("sceneArt").innerHTML="";
 $("sceneText").textContent="";$("sceneSmall").textContent="";
 $("cutscene").classList.remove("active","jitter");
}
function later(fn,t){sceneTimer.push(setTimeout(fn,t))}
function activateScene(){
 clearScene();$("cutscene").classList.add("active");
}
function finishCutscene(){clearScene()}

function addSymbols(chars,count,mode){
 const art=$("sceneArt");
 for(let i=0;i<count;i++){
  const e=document.createElement("div");e.className="symbol";
  e.textContent=chars[Math.floor(Math.random()*chars.length)];
  e.style.left=Math.random()*100+"vw";e.style.top=Math.random()*100+"vh";
  e.style.fontSize=(18+Math.random()*100)+"px";
  e.style.color="hsl("+Math.floor(Math.random()*360)+" 90% 75%)";
  e.style.setProperty("--d",(1.2+Math.random()*3)+"s");
  e.style.setProperty("--x",(-30+Math.random()*60)+"vw");
  e.style.setProperty("--y",(-30+Math.random()*60)+"vh");
  e.style.setProperty("--r",(-180+Math.random()*360)+"deg");
  if(mode)e.style.mixBlendMode=mode;
  art.appendChild(e);
 }
}
function addShapes(count,kind){
 for(let i=0;i<count;i++){
  const e=document.createElement("div");e.className="shape";
  const size=30+Math.random()*240;e.style.width=size+"px";e.style.height=size+"px";
  e.style.left=Math.random()*100+"vw";e.style.top=Math.random()*100+"vh";
  e.style.color="hsl("+Math.random()*360+" 90% 70%)";
  e.style.setProperty("--d",(1+Math.random()*4)+"s");
  if(kind==="triangle")e.style.clipPath="polygon(50% 0,100% 100%,0 100%)";
  if(kind==="diamond")e.style.transform+=" rotate(45deg)";
  $("sceneArt").appendChild(e);
 }
}
function addWords(words,count){
 for(let i=0;i<count;i++){
  const e=document.createElement("div");e.className="word";
  e.textContent=words[Math.floor(Math.random()*words.length)];
  e.style.left=Math.random()*90+"vw";e.style.top=Math.random()*90+"vh";
  e.style.fontSize=(12+Math.random()*55)+"px";
  e.style.color="hsl("+Math.random()*360+" 90% 75%)";
  e.style.setProperty("--d",(.8+Math.random()*2.5)+"s");
  $("sceneArt").appendChild(e);
 }
}
function addLines(count){
 for(let i=0;i<count;i++){
  const e=document.createElement("div");e.className="line";
  e.style.left=Math.random()*100+"vw";e.style.top=Math.random()*100+"vh";
  e.style.width=(40+Math.random()*500)+"px";
  e.style.background="hsl("+Math.random()*360+" 90% 75%)";
  e.style.setProperty("--a",Math.random()*360+"deg");
  e.style.setProperty("--d",(.4+Math.random()*1.8)+"s");
  $("sceneArt").appendChild(e);
 }
}
function text(t,small=""){
 $("sceneText").textContent=t;$("sceneSmall").textContent=small;
}
function bg(css){$("sceneBg").style.background=css}

function playCharacter(c){
 activateScene();
 const scenes={
  drums:()=>{
   bg("radial-gradient(circle,#242424,#080808 55%,#000)");
   text("THE BEAT HAS ARRIVED.");
   addSymbols(["♩","♪","♫","♬","𝄞","∿"],45);
   addShapes(14,"diamond");
   later(()=>text("TRISTAN","DRUMS / IMPACT"),900);
   later(()=>{$("cutscene").classList.add("jitter");addWords(["KICK","SNARE","HIT","TEMPO"],18);audioBeep(110,.7,"square")},1900);
   later(()=>{text("TRISTAN");addShapes(30,"triangle");addSymbols(["●","◉","◌"],30)},3000);
   later(finishCutscene,4700);
  },
  bass:()=>{
   bg("linear-gradient(120deg,#08051b,#21104d,#03030b)");
   text("THE LOW END MOVES FIRST.");
   addSymbols(["∿","⌁","≈","≋","◒","◓"],55,"screen");
   addLines(30);
   later(()=>text("CEA[N]","FREQUENCY DETECTED"),1100);
   later(()=>{addShapes(22,"diamond");addWords(["SUB","LOW","808","VIBRATION"],22);audioBeep(55,.9,"sawtooth")},1900);
   later(()=>text("CEAN");addSymbols(["∿","⌁","≋"],90),3200);
   later(finishCutscene,5000);
  },
  rhythm:()=>{
   bg("conic-gradient(from 20deg,#271100,#050505,#5b2600,#050505)");
   text("KEEP THE TIME.");
   addShapes(18,"triangle");addWords(["1","2","3","4","SYNC","TIME"],35);
   later(()=>text("EVERY STRUM LANDS SOMEWHERE."),1000);
   later(()=>{addSymbols(["/","\\","|","_","⌁","×"],70);addLines(18);audioBeep(190,.5,"square")},1900);
   later(()=>{text("NATHER","RHYTHM LOCKED");$("cutscene").classList.add("jitter");addShapes(45,"diamond")},3000);
   later(finishCutscene,5000);
  },
  voice:()=>{
   bg("radial-gradient(ellipse at center,#4d073d,#11030e 55%,#000)");
   text("A VOICE CUTS THROUGH THE NOISE.");
   addWords(["AH","OO","VOICE","VMA","∞","∿"],45);
   addSymbols(["◡","◠","⌒","∿","≋"],50);
   later(()=>text("THE ROOM STARTS SINGING."),1000);
   later(()=>{addShapes(28,"triangle");addWords(["VOCAL","HARMONY","ECHO"],28);audioBeep(440,.8,"triangle")},2000);
   later(()=>text("JAZLIN");addSymbols(["∿","≈","⌁"],100),3300);
   later(finishCutscene,5200);
  },
  guitar:()=>{
   bg("linear-gradient(135deg,#00131b,#052b3a,#010407)");
   text("ONE STRING. ONE MOMENT.");
   addLines(25);addSymbols(["/","\\","⌁","╱","╲","×"],50);
   later(()=>text("THE STRING STARTS DRAWING."),1000);
   later(()=>{addShapes(25,"diamond");addWords(["AMPLIFY","FEEDBACK","RIFF","SIGNAL"],25);audioBeep(330,.8,"sawtooth")},1900);
   later(()=>{text("LIAM","ELECTRIC GUITAR");$("cutscene").classList.add("jitter");addLines(70)},3100);
   later(finishCutscene,5000);
  },
  legacy:()=>{
   bg("linear-gradient(45deg,#06121d,#16334a,#020507)");
   text("A TRACE WAS LEFT BEHIND.");
   addWords(["ARCHIVE","MEMORY","TAKE","RECORD","01","02","03"],50);
   addSymbols(["□","◇","△","○","∴","∵"],45);
   later(()=>text("OLD SIGNAL. NEW ROOM."),1100);
   later(()=>{addShapes(35,"diamond");addWords(["LEO","PAST","GUITAR","RETURN"],30)},2200);
   later(()=>text("LEO");addSymbols(["□","◇","△"],100),3400);
   later(finishCutscene,5000);
  },
  vocalMemory:()=>{
   bg("radial-gradient(circle,#3a2b13,#0e0a04,#000)");
   text("THE RECORDING NEVER REALLY ENDED.");
   addSymbols(["𝄞","𝄢","♩","♪","∿"],65);
   addWords(["TAKE 1","TAKE 2","REWIND","PLAY","ARCHIVE"],35);
   later(()=>text("THE PAST BLEEDS INTO THE PRESENT."),1200);
   later(()=>{addShapes(30,"triangle");addWords(["EDUARD","VOCAL","MEMORY"],35)},2200);
   later(()=>text("EDUARD");audioBeep(220,.9,"sine"),3400);
   later(finishCutscene,5100);
  },
  tcell:()=>{
   bg("conic-gradient(from 90deg,#03200a,#06110a,#12431b,#020603)");
   text("THE BAND NAME SPLITS.");
   addSymbols(["T","C","E","L","L","⊕","⊗","⌂"],80);
   addShapes(24,"diamond");
   later(()=>text("TCELL IS NOT ONE SHAPE."),1000);
   later(()=>{addWords(["BAND","SIGNAL","MEMBERS","SYNC","TCELL"],50);addLines(40)},1900);
   later(()=>{text("TCELL");$("cutscene").classList.add("jitter");addSymbols(["T","C","E","L","L"],120)},3200);
   later(finishCutscene,5200);
  },
  lookphoria:()=>{
   bg("radial-gradient(circle at center,#80621d 0%,#3b2b0e 18%,#120d04 48%,#000 78%)");
   text("...");
   addSymbols(["◇","◈","✦","✧","✢","⌘","∴","∵","∞"],90);
   addShapes(35,"diamond");
   addWords(["VMA","MEMORY","HISTORY","LOOK","PHORIA"],35);
   later(()=>text("DID YOU HEAR THAT?"),1000);
   later(()=>text("THE LIGHTS ARE FADING."),2000);
   later(()=>{text("VMA HISTORY HAS BEEN UNLOCKED.");addSymbols(["◇","◈","⌘","∞"],120);addLines(45)},3000);
   later(()=>{text("THE SYSTEM IS LOSING CONTROL.");$("cutscene").classList.add("jitter");addShapes(60,"triangle")},4000);
   later(()=>{text("LOOKPHORIA");addWords(["BREAKTHROUGH","LOOKPHORIA","VMA"],70);addSymbols(["◇","◈","∞","∴"],150);audioBeep(70,1,"sawtooth")},5000);
   later(()=>{text("THE VMA REMEMBERS.");$("cutscene").classList.remove("jitter")},6100);
   later(finishCutscene,7600);
  },
  vma:()=>{
   bg("conic-gradient(from 0deg,#fff,#111 8%,#fff 16%,#020202 25%,#fff 34%,#020202 50%,#fff 68%,#000)");
   text("VMA.");
   addSymbols(["V","M","A","∑","∞","◈","⌘","∴","∵","⊕","⊗"],100);
   addShapes(55,"diamond");addWords(["ROLL","VMA","VMA","VMA","REALITY","INDEX","ERROR"],70);
   later(()=>text("THE INDEX HAS NO FINAL ENTRY."),900);
   later(()=>{$("cutscene").classList.add("jitter");addLines(100);addSymbols(["V","M","A","∞"],150);audioBeep(520,1,"square")},1800);
   later(()=>text("YOU DID NOT FIND VMA."),3000);
   later(()=>text("VMA FOUND YOU."),3900);
   later(()=>{text("V M A");addShapes(100,"triangle");addWords(["∞","???","VMA","ROLL","END"],100)},4900);
   later(()=>{$("sceneArt").innerHTML+="<div class='flash'></div>";text("THE END OF THE INDEX IS THE START OF THE NEXT ONE.")},6200);
   later(finishCutscene,8200);
  }
 };
 scenes[c.theme]();
}

function playBreakthrough(n){
 activateScene();
 // Every breakthrough has a different seed/concept, while intensity grows.
 const intensity=Math.min(1,Math.log10(n+1)/8);
 const concepts=[
  ["FRACTURE","◇","□","△"],
  ["MIRRORSPACE","◐","◑","◒"],
  ["OVERWRITE","█","▓","▒","░"],
  ["ORBITAL MAP","⊙","⊕","∴"],
  ["CHROMATIC ERROR","A","B","C","X"],
  ["IMPOSSIBLE PERSPECTIVE","╱","╲","│","─"],
  ["SYMBOL STORM","∑","∞","∂","∇"],
  ["ARCHIVE COLLAPSE","01","10","11","00"],
  ["VMA PARADOX","V","M","A","?"]
 ];
 const concept=concepts[(n-1)%concepts.length];
 bg(`conic-gradient(from ${n*37}deg,#000 0 12%,hsl(${(n*67)%360} 70% 18%),#000 38%,hsl(${(n*113)%360} 80% 12%),#000 70%)`);
 text(`BREAKTHROUGH ${n}`,`ROLL ${state.rolls.toLocaleString()}`);
 addSymbols(concept.slice(1),Math.floor(45+intensity*130),"screen");
 addShapes(Math.floor(15+intensity*80),n%2?"triangle":"diamond");
 addWords([concept[0],"BREAK","SYSTEM","VMA",String(n)],Math.floor(20+intensity*100));
 addLines(Math.floor(10+intensity*100));
 later(()=>{text("REALITY JUST CHANGED SHAPE.");audioBeep(90+Math.min(600,n),.8,"square")},900);
 later(()=>{$("cutscene").classList.add("jitter");addSymbols(["∞","⊗","⊕","∴","∵"],Math.floor(60+intensity*180));addShapes(Math.floor(20+intensity*100),"triangle")},1800);
 later(()=>text(`BREAKTHROUGH ${n}`),3000);
 later(()=>{text(n>=1000?"THE INDEX IS NO LONGER STABLE.":"YOU UNLOCKED SOMETHING.");addWords(["100","200","300","∞","???","VMA"],Math.floor(40+intensity*120))},3900);
 later(()=>{text(`BREAKTHROUGH ${n}`);$("cutscene").classList.remove("jitter");$("sceneArt").innerHTML+="<div class='flash'></div>"},5000);
 later(finishCutscene,6100);
}

updateUI();
</script>
</body>
</html>
'''

path = Path("/mnt/data/roll_a_vma_rewrite.html")
path.write_text(html, encoding="utf-8")
print(f"Created: {path}")


<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Speaking Buddy</title>
<style>
  :root{--bg:#f4f1ff;--card:#fff;--ink:#241b3a;--good:#1fa463;--bad:#e5484d;--accent:#6c4cf1}
  *{box-sizing:border-box}
  body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif;background:var(--bg);color:var(--ink);padding:16px;max-width:560px;margin-inline:auto}
  h1{font-size:20px;margin:4px 0 12px;text-align:center}
  .card{background:var(--card);border-radius:18px;padding:16px;margin-bottom:14px;box-shadow:0 2px 10px rgba(60,40,140,.08)}
  .buddy-wrap{display:flex;align-items:center;gap:12px}
  .buddy{width:84px;height:84px;border-radius:50%;background:#ffd93d;position:relative;flex:none;transition:transform .2s,background .3s;box-shadow:inset -6px -8px 0 rgba(0,0,0,.08)}
  .eye{position:absolute;top:30px;width:12px;height:16px;background:#241b3a;border-radius:50%}
  .eye.l{left:25px}.eye.r{right:25px}
  .mouth{position:absolute;left:50%;bottom:20px;width:30px;height:14px;transform:translateX(-50%);border-bottom:4px solid #241b3a;border-radius:0 0 30px 30px;transition:all .2s}
  .buddy.happy{background:#7be0a3;animation:hop .5s 2}
  .buddy.angry{background:#ff8f8f;animation:shake .4s 2}
  .buddy.angry .mouth{border-bottom:0;border-top:4px solid #241b3a;border-radius:30px 30px 0 0;bottom:18px}
  .buddy.listen{background:#9ad0ff;animation:pulse 1s infinite}
  .buddy.listen .mouth{width:14px;height:14px;border:4px solid #241b3a;border-radius:50%}
  @keyframes hop{50%{transform:translateY(-14px)}}
  @keyframes shake{25%{transform:translateX(-6px)}75%{transform:translateX(6px)}}
  @keyframes pulse{50%{transform:scale(1.08)}}
  .bubble{background:#efeaff;border-radius:14px;padding:10px 12px;font-size:15px;line-height:1.35;flex:1}
  .sample{font-size:18px;line-height:1.7}
  .w{padding:1px 2px;border-radius:4px}
  .w.ok{background:#d9f6e5;color:#0b6b3a}
  .w.miss{background:#ffe0e1;color:#b3262b;text-decoration:underline wavy}
  button{font:inherit;border:0;border-radius:14px;padding:12px 16px;background:var(--accent);color:#fff;font-weight:600;cursor:pointer}
  button.ghost{background:#e8e3ff;color:var(--accent)}
  button:disabled{opacity:.5}
  .row{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
  .rec{background:var(--bad)}
  .stats{display:flex;gap:8px;margin-top:10px}
  .stat{flex:1;background:#f7f5ff;border-radius:12px;padding:8px;text-align:center;font-size:12px}
  .stat b{display:block;font-size:20px}
  select,textarea{font:inherit;width:100%;border-radius:10px;border:1px solid #d6cfff;padding:8px}
  textarea{min-height:110px}
  small{color:#6b6482}
  .hidden{display:none}
  .fix{display:flex;align-items:center;gap:8px;padding:10px 0;border-top:1px solid #eee9ff;flex-wrap:wrap}
  .fix .word{font-size:18px;font-weight:700;min-width:90px}
  .fix .heard{font-size:13px;color:#b3262b;flex:1;min-width:110px}
  .fix .syl{font-size:13px;color:#6b6482}
  .fix button{padding:7px 10px;font-size:13px}
  .fix.done .word{color:var(--good)}
</style>
</head>
<body>
<h1>🎙️ Speaking Buddy</h1>

<div class="card">
  <div class="buddy-wrap">
    <div class="buddy" id="buddy"><div class="eye l"></div><div class="eye r"></div><div class="mouth"></div></div>
    <div class="bubble" id="bubble">Pick a sample, hit record, and read it out loud. I'm listening 👂</div>
  </div>
</div>

<div class="card">
  <div class="row" style="margin-bottom:10px">
    <select id="sampleSel" style="flex:1"></select>
    <select id="spice" style="width:auto">
      <option value="0">😇 Gentle</option>
      <option value="1" selected>😏 Medium</option>
      <option value="2">😈 Savage</option>
    </select>
  </div>
  <div class="sample" id="sample"></div>
  <div class="row" style="margin-top:14px">
    <button id="recBtn">⏺ Record</button>
    <button class="ghost" id="playBtn" disabled>▶ My voice</button>
    <button class="ghost" id="listenBtn">🔊 Hear sample</button>
  </div>
  <div class="stats hidden" id="stats">
    <div class="stat"><b id="sAcc">0%</b>words matched</div>
    <div class="stat"><b id="sWpm">0</b>words / min</div>
    <div class="stat"><b id="sMiss">0</b>missed</div>
  </div>
  <small id="support"></small>
</div>

<div class="card hidden" id="fixCard">
  <b>🛠 Pronunciation fixes</b><br><small>Tap 🔊 to hear the word slowly, then 🎤 to say it. I'll tell you if you nailed it.</small>
  <div id="fixes"></div>
</div>

<div class="card">
  <div class="row" style="justify-content:space-between">
    <b>Teacher mode</b>
    <button class="ghost" id="toggleT">Edit samples</button>
  </div>
  <div id="teacher" class="hidden" style="margin-top:10px">
    <small>One sample per block. Start each with a title line, then the text on the next lines. Separate samples with a line containing only ---</small>
    <textarea id="ta"></textarea>
    <div class="row" style="margin-top:8px"><button id="saveT">Save samples</button></div>
  </div>
</div>#<script>
const DEFAULT = `Speaking Part 1 – Hometown
I live in Tashkent, which is the capital of Uzbekistan. It is a big and busy city with a lot of history. What I like most about my hometown is the food and the friendly people. In the summer it can be extremely hot, but in autumn the weather is lovely.
---
Speaking Part 2 – A person I admire
The person I admire most is my English teacher. She is patient, hardworking and always ready to help. She taught me that mistakes are a natural part of learning, and that is why I never give up.`;

const store = {
  get(){ try{return localStorage.getItem('sb_samples')||DEFAULT}catch(e){return DEFAULT} },
  set(v){ try{localStorage.setItem('sb_samples',v)}catch(e){} }
};

let samples = [], cur = 0, rec = null, chunks = [], audioUrl = null, startedAt = 0, finalText = '', listening = false;

function parse(txt){
  return txt.split(/^\s*---\s*$/m).map(b=>b.trim()).filter(Boolean).map(b=>{
    const lines=b.split('\n'); return {title:lines[0].trim(), text:lines.slice(1).join(' ').trim()||lines[0]};
  });
}
const norm = w => w.toLowerCase().replace(/[^a-z0-9']/g,'');
const $ = id => document.getElementById(id);

function loadSamples(){
  samples = parse(store.get()); $('ta').value = store.get();
  $('sampleSel').innerHTML = samples.map((s,i)=>`<option value="${i}">${s.title}</option>`).join('');
  cur = 0; renderSample();
}
function renderSample(marks){
  const words = samples[cur].text.split(/\s+/);
  $('sample').innerHTML = words.map((w,i)=>`<span class="w ${marks?(marks[i]?'ok':'miss'):''}">${w}</span>`).join(' ');
}

const LINES = {
  great:[ ["Okay, show-off 😎 Nearly perfect. Who recorded this, a robot?","That was clean. I have no roast today… I'm almost disappointed."], ["Wow. Did you memorize this or are you just talented? Either way, respect 👏","Flawless! My circuits are blushing."], ["10/10 🔥 Fine, you win this round. Don't get cocky.","Perfect. Teacher is going to cry happy tears."] ],
  good:[ ["Good! A few words slipped away though. Check the red ones 👀","Nice try! Almost there, polish the red words."], ["Not bad, not bad… but the red words are hiding from you 🙈","Decent! Those red words need a little love."], ["So close! You skipped words like it was a buffet 🍽️","Good, but I heard you mumble. Red words = homework."] ],
  mid:[ ["Hmm. You rushed it. Slow down and say every word 🐢","Half of it made it through. Try once more!"], ["Did you read with your eyes closed? 😅 Many red words!","That was… creative. But not what's on the page."], ["Half the sample vanished. Did it run away from you? 🏃","I've heard better from a sleepy parrot 🦜 Try again!"] ],
  bad:[ ["I could barely catch anything. Check your mic and try again 🎤","That was very quiet or very different. Try again, slowly."], ["Mic check! Is anybody home? 🔔 Try again, louder this time.","I understood almost nothing. Breathe, then read slowly."], ["Was that English or a washing machine? 🧺 Again, please!","My ears are confused. Read it again, word by word."] ]
};
function react(acc, spice){
  const tier = acc>=90?'great':acc>=70?'good':acc>=40?'mid':'bad';
  const arr = LINES[tier][spice]; const msg = arr[Math.floor(Math.random()*arr.length)];
  $('bubble').textContent = msg;
  setMood(acc>=70?'happy':'angry');
}
function setMood(m){ const b=$('buddy'); b.className='buddy '+(m||''); if(m==='happy'||m==='angry') setTimeout(()=>b.className='buddy',1500); }

// LCS alignment of sample words vs spoken words
function align(sampleWords, spokenWords){
  const a=sampleWords.map(norm), b=spokenWords.map(norm);
  const dp=Array.from({length:a.length+1},()=>Array(b.length+1).fill(0));
  for(let i=a.length-1;i>=0;i--)for(let j=b.length-1;j>=0;j--)
    dp[i][j]=a[i]&&a[i]===b[j]?dp[i+1][j+1]+1:Math.max(dp[i+1][j],dp[i][j+1]);
  const marks=Array(a.length).fill(false), matchJ=Array(a.length).fill(-1); let i=0,j=0;
  while(i<a.length&&j<b.length){
    if(a[i]&&a[i]===b[j]){marks[i]=true;matchJ[i]=j;i++;j++}
    else if(dp[i+1][j]>=dp[i][j+1])i++; else j++;
  }
  // what was heard instead of each missed word
  const heard=Array(a.length).fill('');
  for(let k=0;k<a.length;k++){
    if(marks[k]) continue;
    let p=k-1; while(p>=0&&!marks[p])p--;
    let n=k+1; while(n<a.length&&!marks[n])n++;
    const from=p>=0?matchJ[p]+1:0, to=n<a.length?matchJ[n]:spokenWords.length;
    const gap=spokenWords.slice(from,to);
    heard[k]=gap[k-(p+1)]||'';
  }
  return {marks,heard};
}

function syllables(w){
  const s=w.toLowerCase().replace(/[^a-z]/g,'');
  const m=s.match(/[^aeiouy]*[aeiouy]+(?:[^aeiouy]+$|[^aeiouy](?=[^aeiouy]))?|[^aeiouy]+$/g);
  return (m&&m.length?m:[s]).join(' · ');
}
function speak(text, rate){
  if(!('speechSynthesis' in window)){ $('bubble').textContent='This browser can\'t read text aloud. Please open the file in Chrome 🔊'; return; }
  const synth=window.speechSynthesis;
  synth.cancel();
  setTimeout(()=>{
    const u=new SpeechSynthesisUtterance(text);
    const en=synth.getVoices().filter(x=>/^en/i.test(x.lang));
    const pick=en.find(x=>/en[-_]US/i.test(x.lang))||en[0];
    if(pick){ u.voice=pick; u.lang=pick.lang; } else { u.lang='en-US'; }
    u.rate=rate;
    u.onerror=()=>{ $('bubble').textContent='No sound? Turn the volume up, switch off silent mode, and open this file in Chrome 🔊'; };
    synth.resume(); synth.speak(u);
  },120);
}
if('speechSynthesis' in window){ speechSynthesis.getVoices(); speechSynthesis.onvoiceschanged=()=>speechSynthesis.getVoices(); }
function sayWord(w){ speak(w,.6); }
function tryWord(w,row){
  if(!SR){ row.querySelector('.heard').textContent='Speech check not supported here.'; return; }
  const r=new SR(); r.lang='en-US'; r.maxAlternatives=3;
  const out=row.querySelector('.heard'); out.style.color='#6b6482'; out.textContent='Listening… 👂'; setMood('listen');
  r.onresult=e=>{
    const alts=[...e.results[0]].map(x=>norm(x.transcript));
    const said=e.results[0][0].transcript.trim();
    setMood('');
    if(alts.includes(norm(w))){ row.classList.add('done'); out.style.color='#1fa463'; out.textContent='✅ Perfect!'; $('bubble').textContent='"'+w+'" fixed! Look at you 💪'; setMood('happy'); }
    else { out.style.color='#b3262b'; out.textContent='I heard "'+said+'". Listen again 🔊'; $('bubble').textContent='Almost! Say "'+w+'" slowly 🐢'; }
  };
  r.onerror=()=>{ setMood(''); out.textContent='Couldn\'t hear you. Try again 🎤'; };
  r.onend=()=>setMood('');
  try{r.start()}catch(e){}
}
function renderFixes(sw,marks,heard){
  const list=[]; const seen=new Set();
  sw.forEach((w,i)=>{ const c=w.replace(/[^A-Za-z']/g,''); if(!marks[i]&&c&&!seen.has(c.toLowerCase())){seen.add(c.toLowerCase());list.push({c,h:heard[i]});} });
  const box=$('fixes'); box.innerHTML='';
  list.slice(0,12).forEach(({c,h})=>{
    const row=document.createElement('div'); row.className='fix';
    row.innerHTML=`<span class="word">${c}</span><span class="syl">${syllables(c)}</span><span class="heard">${h?'I heard: "'+h+'"':'I didn\'t catch this word'}</span><button class="ghost">🔊</button><button>🎤</button>`;
    const [b1,b2]=row.querySelectorAll('button');
    b1.onclick=()=>sayWord(c); b2.onclick=()=>tryWord(c,row);
    box.appendChild(row);
  });
  $('fixCard').classList.toggle('hidden',list.length===0);
} // speech recognition
const SR = window.SpeechRecognition||window.webkitSpeechRecognition;
let recog=null;
if(!SR) $('support').textContent = '⚠️ Speech recognition needs Chrome (Android/desktop) or Safari. Your browser may not support it.';
else $('support').textContent = 'Tip: allow the microphone when asked. Works best in Chrome.';

async function startRec(){
  finalText=''; chunks=[]; $('stats').classList.add('hidden'); $('fixCard').classList.add('hidden'); renderSample();
  try{
    const stream = await navigator.mediaDevices.getUserMedia({audio:true});
    rec = new MediaRecorder(stream);
    rec.ondataavailable = e=>chunks.push(e.data);
    rec.onstop = ()=>{ stream.getTracks().forEach(t=>t.stop()); if(audioUrl)URL.revokeObjectURL(audioUrl); audioUrl=URL.createObjectURL(new Blob(chunks)); $('playBtn').disabled=false; };
    rec.start();
  }catch(e){ $('bubble').textContent='I need your microphone to hear you 🎤 Please allow access and try again.'; return; }
  if(SR){
    recog=new SR(); recog.lang='en-US'; recog.continuous=true; recog.interimResults=false;
    recog.onresult=e=>{ for(let i=e.resultIndex;i<e.results.length;i++) if(e.results[i].isFinal) finalText+=' '+e.results[i][0].transcript; };
    recog.onend=()=>{ if(listening) try{recog.start()}catch(e){} };
    try{recog.start()}catch(e){}
  }
  startedAt=Date.now(); listening=true;
  $('recBtn').textContent='⏹ Stop'; $('recBtn').classList.add('rec'); setMood('listen');
  $('bubble').textContent='Listening… read the whole text 👂';
}
function stopRec(){
  listening=false; if(recog) try{recog.stop()}catch(e){}
  if(rec&&rec.state!=='inactive') rec.stop();
  $('recBtn').textContent='⏺ Record'; $('recBtn').classList.remove('rec'); setMood('');
  $('bubble').textContent='Checking… 🤔';
  const secs=(Date.now()-startedAt)/1000;
  setTimeout(()=>evaluate(secs), 900);
}
function evaluate(secs){
  const sw=samples[cur].text.split(/\s+/), spoken=finalText.trim().split(/\s+/).filter(Boolean);
  if(!SR){ $('bubble').textContent='Your browser can\'t check speech, but you can replay your voice and compare with the sample 🎧'; return; }
  const {marks,heard}=align(sw,spoken); renderSample(marks); renderFixes(sw,marks,heard);
  const ok=marks.filter(Boolean).length, acc=Math.round(ok/sw.length*100);
  const wpm=secs>1?Math.round(spoken.length/secs*60):0;
  $('sAcc').textContent=acc+'%'; $('sWpm').textContent=wpm; $('sMiss').textContent=sw.length-ok;
  $('stats').classList.remove('hidden');
  react(spoken.length<3?0:acc, +$('spice').value);
}

$('recBtn').onclick=()=>listening?stopRec():startRec();
$('playBtn').onclick=()=>{ if(audioUrl) new Audio(audioUrl).play(); };
$('listenBtn').onclick=()=>speak(samples[cur].text,.9);
$('sampleSel').onchange=e=>{ cur=+e.target.value; renderSample(); $('stats').classList.add('hidden'); $('fixCard').classList.add('hidden'); $('bubble').textContent='New sample! Hit record when ready 🎙️'; };
$('toggleT').onclick=()=>$('teacher').classList.toggle('hidden');
$('saveT').onclick=()=>{ store.set($('ta').value); loadSamples(); $('bubble').textContent='Samples saved ✅'; };
loadSamples();
</script>
</body>
</html>Buddy-
AI speaking buddy for English learners. Students read teacher samples aloud and get instant pronunciation feedback.

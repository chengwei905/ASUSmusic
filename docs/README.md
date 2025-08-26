<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Music Player - /music/*.mp3</title>
  <script src="https://cdn.jsdelivr.net/npm/music-metadata-browser/dist/music-metadata-browser.umd.min.js"></script>
  <style>
    /* --- 保留第一版的樣式 --- */
    :root {
      --bg:#0b1020; --card:#121a35; --muted:#a7b0c3; --txt:#e9eefc; --accent:#7aa2ff;
      --radius:16px;
    }
    *{box-sizing:border-box}
    body{
      margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial;
      background: radial-gradient(1200px 600px at 70% -10%, #1a2758, transparent 60%), var(--bg);
      color:var(--txt);
    }
    .wrap{ max-width:960px; margin:40px auto; padding:0 16px; }
    .card{ background:linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border:1px solid rgba(255,255,255,.08); border-radius:var(--radius);
      box-shadow:0 8px 30px rgba(0,0,0,.25); padding:20px; }
    h1{font-size:24px; margin:0 0 12px}
    .row{display:flex; gap:14px; flex-wrap:wrap; align-items:center}
    select,button,input[type="range"]{
      background:#0e1630; color:var(--txt); border:1px solid rgba(255,255,255,.12);
      border-radius:12px; padding:10px 12px; outline:none; font-size:14px;
    }
    button{cursor:pointer}
    button.primary{background:var(--accent); color:#0a0f22; border:none; font-weight:700}
    button.ghost{background:#0e1630}
    .meta{display:grid; grid-template-columns:120px 1fr; gap:12px; margin-top:18px}
    .cover{width:120px; height:120px; border-radius:12px; overflow:hidden; background:#0e1630;
      display:flex; align-items:center; justify-content:center; font-size:12px; color:var(--muted)}
    .fields{display:grid; grid-template-columns:repeat(2, minmax(180px,1fr)); gap:10px}
    .field{background:#0f1732; border:1px solid rgba(255,255,255,.08); border-radius:10px; padding:10px}
    .label{font-size:12px; color:var(--muted)}
    .value{font-size:14px; margin-top:2px; word-break:break-word}
    .controls{display:flex; align-items:center; gap:10px; margin-top:14px}
    .time{min-width:62px; text-align:center; font-variant-tabular-nums:tabular-nums}
    .hint{color:var(--muted); font-size:13px; margin-top:10px}
    .spacer{flex:1}
    .range{flex:1; display:flex; align-items:center; gap:8px}
    input[type="range"]{width:100%}
    .bar{height:6px; background:#0e1630; border-radius:999px; position:relative; overflow:hidden}
    .bar > span{position:absolute; inset:0 100% 0 0; background:linear-gradient(90deg,#7aa2ff,#9ec4ff)}
    @media (max-width:640px){ .meta{grid-template-columns:1fr} .fields{grid-template-columns:1fr} }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>🎧 /music/*.mp3 播放器</h1>

      <div class="row">
        <label for="trackSelect">播放清單：</label>
        <select id="trackSelect"></select>
        <button id="btnPrev" class="ghost">⏮ 上一首</button>
        <button id="btnPlay" class="primary">▶ 播放</button>
        <button id="btnNext" class="ghost">⏭ 下一首</button>
        <button id="btnLoop" class="ghost" title="循環播放">🔁 循環關</button>
        <button id="btnShuffle" class="ghost" title="隨機播放">🔀 隨機關</button>
        <span class="spacer"></span>
        <label class="range">音量
          <input id="volume" type="range" min="0" max="1" step="0.01" value="1">
        </label>
      </div>

      <div class="controls">
        <div class="time" id="curTime">0:00</div>
        <div class="bar" style="flex:1"><span id="progress"></span></div>
        <div class="time" id="durTime">0:00</div>
      </div>

      <div class="meta">
        <div class="cover" id="cover">封面</div>
        <div class="fields" id="fields"></div>
      </div>

      <div class="hint" id="status">正在載入 /music/ 內的 mp3 清單…</div>
      <audio id="audio" preload="metadata" crossorigin="anonymous"></audio>
    </div>
  </div>

<script>
(function(){
  const MUSIC_BASE = '/music/'; // mp3 目錄
  const mm = window.musicMetadataBrowser;
  let playlist = [];
  let idx = 0, shuffle=false, loop=false;

  const audio = document.getElementById('audio');
  const sel = document.getElementById('trackSelect');
  const btnPlay = document.getElementById('btnPlay');
  const btnPrev = document.getElementById('btnPrev');
  const btnNext = document.getElementById('btnNext');
  const btnLoop = document.getElementById('btnLoop');
  const btnShuffle = document.getElementById('btnShuffle');
  const volume = document.getElementById('volume');
  const status = document.getElementById('status');
  const cover = document.getElementById('cover');
  const fields = document.getElementById('fields');
  const curTime = document.getElementById('curTime');
  const durTime = document.getElementById('durTime');
  const progress = document.getElementById('progress');

  const fmtTime = s=>!isFinite(s)?'0:00':`${Math.floor(s/60)}:${Math.floor(s%60).toString().padStart(2,'0')}`;
  const cleanName = n=>decodeURIComponent(n).replace(/[_-]+/g,' ').trim();

  function setButtons(){
    btnPlay.textContent = audio.paused ? '▶ 播放' : '⏸ 暫停';
    btnLoop.textContent = loop ? '🔁 循環開' : '🔁 循環關';
    btnShuffle.textContent = shuffle ? '🔀 隨機開' : '🔀 隨機關';
  }
  function populateSelect(){
    sel.innerHTML='';
    playlist.forEach((t,i)=>{
      const opt=document.createElement('option');
      opt.value=i; opt.textContent=`${(i+1).toString().padStart(2,'0')} • ${t.name}`;
      sel.appendChild(opt);
    });
    sel.value=String(idx);
  }
  function clearMeta(){ cover.textContent='封面'; cover.style.backgroundImage=''; fields.innerHTML=''; }
  function putField(label,value){ const el=document.createElement('div'); el.className='field'; el.innerHTML=`<div class="label">${label}</div><div class="value">${value||'—'}</div>`; fields.appendChild(el); }

  async function loadTrack(i){
    idx=i; sel.value=String(i);
    const item=playlist[i]; clearMeta();
    audio.src=item.url; audio.play().catch(()=>{}); setButtons();

    try{
      const resp=await fetch(item.url); const blob=await resp.blob();
      const metadata=await mm.parseBlob(blob);
      const c=metadata.common||{}, f=metadata.format||{};
      if(c.picture&&c.picture[0]){
        const blobImg=new Blob([new Uint8Array(c.picture[0].data)],{type:c.picture[0].format});
        const url=URL.createObjectURL(blobImg);
        cover.innerHTML=`<img src="${url}" style="width:100%;height:100%;object-fit:cover">`;
      }
      putField('標題',c.title||item.name);
      putField('演出者',c.artist||'');
      putField('專輯',c.album||'');
      putField('年份',c.year||'');
      putField('流派',(c.genre||[]).join(', '));
      putField('曲序',c.track?.no||'');
      putField('長度',f.duration?fmtTime(f.duration):'');
    }catch(e){ status.textContent='無法解析 ID3'; }
  }

  function nextIndex(){
    if(shuffle){ let r; do{r=Math.floor(Math.random()*playlist.length);}while(r===idx); return r; }
    return (idx+1)%playlist.length;
  }

  btnPlay.onclick=()=>audio.paused?audio.play():audio.pause();
  btnPrev.onclick=()=>playlist.length&&loadTrack((idx-1+playlist.length)%playlist.length);
  btnNext.onclick=()=>playlist.length&&loadTrack(nextIndex());
  btnLoop.onclick=()=>{loop=!loop;setButtons();};
  btnShuffle.onclick=()=>{shuffle=!shuffle;setButtons();};
  volume.oninput=()=>audio.volume=+volume.value;
  sel.onchange=()=>loadTrack(+sel.value);
  audio.onplay=setButtons; audio.onpause=setButtons;
  audio.onended=()=>loop?(audio.currentTime=0,audio.play()):loadTrack(nextIndex());
  audio.ontimeupdate=()=>{
    curTime.textContent=fmtTime(audio.currentTime);
    durTime.textContent=fmtTime(audio.duration||0);
    const p=audio.duration?(audio.currentTime/audio.duration):0;
    progress.style.right=(100-Math.min(100,Math.max(0,p*100)))+'%';
  };

  // 🚩 啟動流程：直接掃描 /music/ 目錄
  async function init(){
    try{
      const resp=await fetch(MUSIC_BASE);
      const html=await resp.text();
      const files=[...html.matchAll(/href="([^"]+\.mp3)"/gi)].map(m=>m[1]).sort();
      playlist=files.map(f=>({name:cleanName(f.split('/').pop()),url:MUSIC_BASE+f}));
      if(!playlist.length){status.textContent='未找到 mp3 檔案';return;}
      populateSelect(); loadTrack(0);
      status.textContent='已載入 '+playlist.length+' 首歌曲。';
    }catch(e){ status.textContent='⚠ 無法讀取 /music/ 目錄，請確認伺服器已開啟 autoindex。'; }
  }
  init();
})();
</script>
</body>
</html>




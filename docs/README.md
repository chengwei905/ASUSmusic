<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Music Player - /music/*.mp3</title>
  <script src="https://cdn.jsdelivr.net/npm/music-metadata-browser/dist/music-metadata-browser.umd.min.js"></script>
  <style>
    body{font-family:system-ui; background:#111; color:#eee; margin:0; padding:20px;}
    .player{max-width:800px; margin:auto;}
    select,button,input[type="range"]{margin:5px; padding:5px;}
    .meta{margin-top:15px;}
    .cover{width:150px; height:150px; background:#333; display:flex; align-items:center; justify-content:center;}
  </style>
</head>
<body>
<div class="player">
  <h2>🎵 /music 播放器</h2>
  <div>
    <button id="btnPrev">⏮ 上一首</button>
    <button id="btnPlay">▶ 播放</button>
    <button id="btnNext">⏭ 下一首</button>
    <label>清單: <select id="trackSelect"></select></label>
  </div>
  <audio id="audio" preload="metadata"></audio>
  <div class="meta">
    <div class="cover" id="cover">封面</div>
    <div id="info"></div>
  </div>
</div>

<script>
const MUSIC_BASE = '/music/';
const mm = window.musicMetadataBrowser;
let playlist = [];
let idx = 0;

const audio = document.getElementById('audio');
const btnPlay = document.getElementById('btnPlay');
const btnPrev = document.getElementById('btnPrev');
const btnNext = document.getElementById('btnNext');
const sel = document.getElementById('trackSelect');
const cover = document.getElementById('cover');
const info = document.getElementById('info');

function cleanName(name){ return decodeURIComponent(name).replace(/[_-]+/g,' '); }

async function loadTrack(i){
  idx = i;
  sel.value = String(i);
  const item = playlist[i];
  audio.src = item.url;
  audio.play().catch(()=>{});
  btnPlay.textContent = '⏸ 暫停';

  cover.textContent = '封面'; cover.style.background='';
  info.textContent = '載入中…';

  try{
    const resp = await fetch(item.url);
    const blob = await resp.blob();
    const metadata = await mm.parseBlob(blob);
    const c = metadata.common || {};
    if(c.picture && c.picture[0]){
      const blobImg = new Blob([new Uint8Array(c.picture[0].data)], {type:c.picture[0].format});
      const url = URL.createObjectURL(blobImg);
      cover.innerHTML = `<img src="${url}" style="width:100%;height:100%;object-fit:cover">`;
    }
    info.innerHTML = `
      標題: ${c.title||item.name}<br>
      專輯: ${c.album||''}<br>
      演出者: ${(c.artist||'')}<br>
      年份: ${c.year||''}
    `;
  }catch(e){ info.textContent = '無法解析 ID3 標籤'; }
}

function populateSelect(){
  sel.innerHTML='';
  playlist.forEach((t,i)=>{
    const opt=document.createElement('option');
    opt.value=i; opt.textContent=(i+1)+'. '+t.name;
    sel.appendChild(opt);
  });
  sel.value=0;
}

btnPlay.onclick=()=>{
  if(audio.paused){ audio.play(); btnPlay.textContent='⏸ 暫停'; }
  else{ audio.pause(); btnPlay.textContent='▶ 播放'; }
};
btnPrev.onclick=()=>{ if(playlist.length) loadTrack((idx-1+playlist.length)%playlist.length); };
btnNext.onclick=()=>{ if(playlist.length) loadTrack((idx+1)%playlist.length); };
sel.onchange=()=>{ loadTrack(Number(sel.value)); };
audio.onended=()=>{ loadTrack((idx+1)%playlist.length); };

async function init(){
  try{
    const resp = await fetch(MUSIC_BASE);
    const html = await resp.text();
    const matches = [...html.matchAll(/href="([^"]+\.mp3)"/gi)];
    const files = matches.map(m=>m[1]).sort();
    playlist = files.map(f=>({name:cleanName(f.split('/').pop()), url:MUSIC_BASE+f}));
    if(playlist.length===0){ info.textContent='未找到 mp3 檔案'; return; }
    populateSelect();
    // 預設載入第一首，但不自動播放，要等使用者按 Play
    loadTrack(0);
    audio.pause();
    btnPlay.textContent='▶ 播放';
  }catch(e){
    info.textContent='無法讀取 /music/ 目錄，請確認伺服器已開啟 autoindex。';
  }
}

init();
</script>
</body>
</html>



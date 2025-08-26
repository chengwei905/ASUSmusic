<html lang="zh-Hant">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Music Player - /music/*.mp3</title>
  <!-- 解析音訊標籤：music-metadata-browser（純前端） -->
  <script src="https://cdn.jsdelivr.net/npm/music-metadata-browser/dist/music-metadata-browser.umd.min.js"></script>
  <style>
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
    .wrap{
      max-width:960px; margin:40px auto; padding:0 16px;
    }
    .card{
      background:linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border:1px solid rgba(255,255,255,.08);
      border-radius:var(--radius); box-shadow:0 8px 30px rgba(0,0,0,.25);
      padding:20px;
    }
    h1{font-size:24px; margin:0 0 12px}
    .row{display:flex; gap:14px; flex-wrap:wrap; align-items:center}
    select,button,input[type="range"]{
      background:#0e1630; color:var(--txt); border:1px solid rgba(255,255,255,.12);
      border-radius:12px; padding:10px 12px; outline:none; font-size:14px;
    }
    button{cursor:pointer}
    button.primary{background:var(--accent); color:#0a0f22; border:none; font-weight:700}
    button.ghost{background:#0e1630}
    .meta{
      display:grid; grid-template-columns:120px 1fr; gap:12px; margin-top:18px
    }
    .cover{
      width:120px; height:120px; border-radius:12px; overflow:hidden; background:#0e1630;
      display:flex; align-items:center; justify-content:center; font-size:12px; color:var(--muted)
    }
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
    .folder-picker{margin-top:10px}
    .error{color:#ffb3b3}
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
        <div class="bar" style="flex:1">
          <span id="progress"></span>
        </div>
        <div class="time" id="durTime">0:00</div>
      </div>

      <div class="meta">
        <div class="cover" id="cover">封面</div>
        <div class="fields" id="fields">
          <!-- 動態填入 -->
        </div>
      </div>

      <div class="hint" id="status">嘗試從 <code>/music/playlist.json</code> 或目錄索引載入播放清單…</div>

      <div class="folder-picker">
        <button id="pickFolder" class="ghost">或選擇本機資料夾播放…</button>
        <span class="hint">（需要新式瀏覽器與 HTTPS）</span>
      </div>

      <audio id="audio" preload="metadata" crossorigin="anonymous"></audio>
    </div>
  </div>

<script>
(function(){
  const MUSIC_BASE = '/music/'; // 你的 mp3 目錄
  const SUPPORTED_EXT = ['.mp3', '.m4a', '.aac', '.flac', '.ogg', '.wav']; // 也支援其他格式（若瀏覽器支援）
  const mm = window.musicMetadataBrowser;

  /*** 狀態 ***/
  let playlist = [];         // [{name, url, file?}]
  let idx = 0;
  let shuffle = false;
  let loop = false;
  let usingLocalFS = false;  // 是否使用本機資料夾
  let directoryHandle = null;

  /*** 元件 ***/
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
  const pickFolder = document.getElementById('pickFolder');

  /*** 工具 ***/
  const fmtTime = s => {
    if (!isFinite(s)) return '0:00';
    const m = Math.floor(s / 60);
    const r = Math.floor(s % 60);
    return `${m}:${r.toString().padStart(2,'0')}`;
  };
  const extOf = name => (name.match(/\.[a-z0-9]+$/i)||[''])[0].toLowerCase();
  const isAudioFile = name => SUPPORTED_EXT.includes(extOf(name));
  const cleanName = name => decodeURIComponent(name).replace(/[_-]+/g,' ').replace(/\s+/g,' ').trim();

  function setStatus(msg, isError=false){
    status.innerHTML = isError ? `<span class="error">${msg}</span>` : msg;
  }
  function setButtons(){
    btnPlay.textContent = audio.paused ? '▶ 播放' : '⏸ 暫停';
    btnLoop.textContent = loop ? '🔁 循環開' : '🔁 循環關';
    btnShuffle.textContent = shuffle ? '🔀 隨機開' : '🔀 隨機關';
  }
  function populateSelect(){
    sel.innerHTML = '';
    playlist.forEach((t, i)=>{
      const opt = document.createElement('option');
      opt.value = i;
      opt.textContent = `${(i+1).toString().padStart(2,'0')} • ${t.name}`;
      sel.appendChild(opt);
    });
    sel.value = String(idx);
  }
  function setCoverFromPicture(picture){
    if(!picture || !picture.data) { cover.textContent='無封面'; cover.style.backgroundImage=''; return; }
    const blob = new Blob([new Uint8Array(picture.data)], {type: picture.format || 'image/jpeg'});
    const url = URL.createObjectURL(blob);
    cover.innerHTML = `<img alt="cover" src="${url}" style="width:100%;height:100%;object-fit:cover" />`;
  }
  function putField(label, value){
    const el = document.createElement('div');
    el.className='field';
    el.innerHTML = `<div class="label">${label}</div><div class="value">${value || '—'}</div>`;
    fields.appendChild(el);
  }
  function clearMeta(){
    cover.textContent='封面';
    cover.style.backgroundImage='';
    fields.innerHTML='';
  }

  /*** 載入/解析標籤 ***/
  async function loadTrack(i){
    idx = i;
    sel.value = String(i);
    const item = playlist[i];
    clearMeta();

    try{
      let src, displayName = item.name;
      let blob = null;

      if (item.file) { // 本機檔案
        blob = item.file;
        src = URL.createObjectURL(blob);
      } else { // 遠端 URL
        src = item.url;
        const resp = await fetch(src);
        blob = await resp.blob(); // 供 metadata 解析
      }

      audio.src = src;
      audio.play().catch(()=>{}); // 可能被瀏覽器阻擋，按播放鍵即可
      setButtons();

      // 解析 ID3
      const metadata = await mm.parseBlob(blob).catch(()=>null);
      const common = metadata?.common || {};
      const format = metadata?.format || {};

      // 封面
      setCoverFromPicture(common.picture?.[0]);

      // 顯示欄位
      const title  = common.title || displayName;
      const artist = (common.artists && common.artists.join(', ')) || common.artist || '';
      const album  = common.album || '';
      const year   = common.year || '';
      const genre  = (common.genre && common.genre.join(', ')) || '';
      const track  = (common.track && (common.track.no + (common.track.of ? ` / ${common.track.of}`:''))) || '';
      const dur    = format.duration ? fmtTime(format.duration) : '';

      putField('標題', title);
      putField('演出者', artist);
      putField('專輯', album);
      putField('年份', year);
      putField('曲序', track);
      putField('流派', genre);
      putField('長度', dur);
      putField('編碼/位元率', [format.codec, format.sampleRate?`${format.sampleRate} Hz`:null, format.bitrate?`${Math.round(format.bitrate/1000)} kbps`:null].filter(Boolean).join(' • '));

      setStatus(usingLocalFS ? '使用本機資料夾播放中。' : `來源：${src}`);
    }catch(err){
      console.error(err);
      setStatus('載入或解析失敗，請檢查檔案與權限。', true);
    }
  }

  /*** 自動下一首 / 控制 ***/
  function nextIndex(){
    if (shuffle) {
      if (playlist.length <= 1) return idx;
      let r;
      do { r = Math.floor(Math.random()*playlist.length); } while (r===idx);
      return r;
    }
    return (idx + 1) % playlist.length;
  }

  /*** 初始化播放清單（遠端） ***/
  async function tryLoadFromPlaylistJson(){
    const url = MUSIC_BASE + 'playlist.json?_=' + Date.now();
    const resp = await fetch(url, {cache:'no-store'});
    if (!resp.ok) throw new Error('no playlist.json');
    const list = await resp.json();
    if (!Array.isArray(list) || list.length===0) throw new Error('empty playlist.json');
    playlist = list.filter(x=>typeof x==='string' && isAudioFile(x))
                   .map(name=>({ name: cleanName(name.replace(/^.*\//,'')), url: MUSIC_BASE + name.replace(/^\.\//,'') }));
    if (playlist.length===0) throw new Error('no audio in playlist.json');
    setStatus('已從 playlist.json 載入播放清單。');
  }

  async function tryLoadFromAutoIndex(){
    const resp = await fetch(MUSIC_BASE, {cache:'no-store'});
    if (!resp.ok) throw new Error('autoindex not available');
    const html = await resp.text();
    // 簡單解析 <a href="xxx.mp3"> 連結（Apache/Nginx autoindex 通常可行）
    const hrefs = Array.from(html.matchAll(/href="([^"]+\.(?:mp3|m4a|aac|flac|ogg|wav))"/gi)).map(m=>m[1]);
    const uniq = [...new Set(hrefs)];
    const files = uniq.filter(isAudioFile);
    if (files.length===0) throw new Error('no audio links found');
    playlist = files.map(h=>{
      const url = h.startsWith('http') ? h : (MUSIC_BASE + h.replace(/^\.\//,''));
      const name = cleanName(url.split('/').pop());
      return { name, url };
    });
    setStatus('已從目錄索引自動建立播放清單。');
  }

  /*** 本機資料夾 ***/
  async function pickLocalFolder(){
    if (!window.showDirectoryPicker){
      setStatus('此瀏覽器不支援本機資料夾選擇（需 Chromium/Edge/Chrome，且 HTTPS）。', true);
      return;
    }
    try{
      directoryHandle = await window.showDirectoryPicker({mode:'read'});
      const files = [];
      for await (const [name, handle] of directoryHandle.entries()){
        if (handle.kind === 'file' && isAudioFile(name)){
          files.push({name, handle});
        }
      }
      files.sort((a,b)=>a.name.localeCompare(b.name, undefined, {numeric:true,sensitivity:'base'}));
      if (files.length===0){ setStatus('資料夾內沒有可播放的音訊檔。', true); return; }

      usingLocalFS = true;
      playlist = [];
      for (const f of files){
        const file = await (await f.handle.getFile());
        playlist.push({ name: cleanName(f.name.replace(/\.[^.]+$/,'')), url: URL.createObjectURL(file), file });
      }
      populateSelect();
      await loadTrack(0);
      setButtons();
    }catch(err){
      if (err?.name !== 'AbortError'){
        console.error(err);
        setStatus('讀取本機資料夾失敗：' + err.message, true);
      }
    }
  }

  /*** 綁定事件 ***/
  btnPlay.addEventListener('click', ()=> audio.paused ? audio.play() : audio.pause());
  btnPrev.addEventListener('click', ()=> { if (playlist.length){ const ni = (idx - 1 + playlist.length) % playlist.length; loadTrack(ni); } });
  btnNext.addEventListener('click', ()=> { if (playlist.length){ loadTrack( nextIndex() ); } });
  btnLoop.addEventListener('click', ()=> { loop = !loop; setButtons(); });
  btnShuffle.addEventListener('click', ()=> { shuffle = !shuffle; setButtons(); });
  volume.addEventListener('input', ()=> audio.volume = Number(volume.value));
  sel.addEventListener('change', ()=> { const i = Number(sel.value); if (!Number.isNaN(i)) loadTrack(i); });
  pickFolder.addEventListener('click', pickLocalFolder);

  audio.addEventListener('play', setButtons);
  audio.addEventListener('pause', setButtons);
  audio.addEventListener('ended', ()=>{
    if (loop){ audio.currentTime = 0; audio.play(); return; }
    if (!playlist.length) return;
    loadTrack( nextIndex() );
  });
  audio.addEventListener('timeupdate', ()=>{
    curTime.textContent = fmtTime(audio.currentTime);
    durTime.textContent = fmtTime(audio.duration || 0);
    const p = audio.duration ? (audio.currentTime / audio.duration) : 0;
    progress.style.right = (100 - Math.min(100, Math.max(0, p*100))) + '%';
  });

  /*** 啟動流程 ***/
  (async function init(){
    try{
      await tryLoadFromPlaylistJson();
    }catch(e1){
      try{
        await tryLoadFromAutoIndex();
      }catch(e2){
        setStatus('未找到 playlist.json，且伺服器未開啟目錄索引。你仍可點「或選擇本機資料夾播放…」。', true);
      }
    }
    if (playlist.length){
      populateSelect();
      await loadTrack(0);
    }
    setButtons();
  })();

})();
</script>
</body>
</html>


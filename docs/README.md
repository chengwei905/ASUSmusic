<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>音樂播放器（含 metadata）</title>
  <script src="https://cdn.jsdelivr.net/npm/jsmediatags@3.9.7/dist/jsmediatags.min.js"></script>
  <style>
    body {
      font-family: DFKai-SB,sans-serif;
      background-color: #c0f4f4;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
    }
    #player-container {
      background-color: #00a0f4;
      padding: 20px;
      border-radius: 20px;
      width: 100%;
      max-width: 400px;
      text-align: center;
      color: white;
    }
    select, audio, input {
      width: 100%;
      margin: 10px 0;
    }
    #metadata p {
      margin: 5px 0;
    }
  </style>
</head>
<body>
  <div id="player-container">
    <h1>🎶音樂播放器</h1>
    <h2 id="media-title">請選擇曲目</h2>

    <div id="metadata">
      <p align="left"><strong>標題:</strong> <span id="meta-title">-</span></p>
      <p align="left"><strong>專輯:</strong> <span id="meta-album">-</span></p>
      <p align="left"><strong>藝術家:</strong> <span id="meta-artist">-</span></p>
    </div>

    <select id="track-selector"></select>
    <audio id="audio-player" controls></audio>
  </div>

  <script>
    const trackSelector = document.getElementById('track-selector');
    const audioPlayer = document.getElementById('audio-player');
    const mediaTitle = document.getElementById('media-title');

    const metaTitle = document.getElementById('meta-title');
    const metaAlbum = document.getElementById('meta-album');
    const metaArtist = document.getElementById('meta-artist');

    let tracks = [];

    function loadTracks() {
      fetch("tracks/tracks.json")
        .then(res => res.json())
        .then(data => {
          tracks = data;
          trackSelector.innerHTML = "";

          tracks.forEach((file, i) => {
            const option = document.createElement('option');
            option.value = file;
            option.textContent = file;
            trackSelector.appendChild(option);
          });

          if (tracks.length > 0) {
            playTrack(tracks[0]);
            trackSelector.value = tracks[0];
          }
        })
        .catch(err => {
          console.error("無法載入 tracks.json:", err);
        });
    }

    function playTrack(file) {
      const filePath = `tracks/${file}`;
      audioPlayer.src = filePath;
      audioPlayer.play();

      mediaTitle.textContent = file;
      metaTitle.textContent = ' ';
      metaAlbum.textContent = ' ';
      metaArtist.textContent = ' ';

      fetch(filePath)
        .then(res => res.blob())
        .then(blob => {
          jsmediatags.read(blob, {
            onSuccess: tag => {
              const tags = tag.tags;
              metaTitle.textContent = tags.title || '(無標題)';
              metaAlbum.textContent = tags.album || '(無專輯)';
              metaArtist.textContent = tags.artist || '(無藝術家)';
              if (tags.title) mediaTitle.textContent = tags.title;
            },
            onError: err => console.error("Metadata 讀取失敗:", err)
          });
        });
    }

    trackSelector.addEventListener('change', e => {
      playTrack(e.target.value);
    });

    audioPlayer.addEventListener('ended', () => {
      const current = trackSelector.value;
      const idx = tracks.indexOf(current);
      if (idx >= 0 && idx + 1 < tracks.length) {
        const next = tracks[idx + 1];
        trackSelector.value = next;
        playTrack(next);
      }
    });

    loadTracks();
  </script>
</body>
</html>









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

    <input type="file" id="file-input" accept="audio/mp3" multiple />

    <div id="metadata">
      <p align="left"><strong>標題:</strong> <span id="meta-title">-</span></p>
      <p align="left"><strong>專輯:</strong> <span id="meta-album">-</span></p>
      <p align="left"><strong>藝術家:</strong> <span id="meta-artist">-</span></p>
    </div>

    <select id="track-selector"></select>
    <audio id="audio-player" controls></audio>
  </div>

  <script>
    const fileInput = document.getElementById("file-input");
    const trackSelector = document.getElementById('track-selector');
    const audioPlayer = document.getElementById('audio-player');
    const mediaTitle = document.getElementById('media-title');

    const metaTitle = document.getElementById('meta-title');
    const metaAlbum = document.getElementById('meta-album');
    const metaArtist = document.getElementById('meta-artist');

    let tracks = [];

    fileInput.addEventListener("change", (e) => {
      const files = Array.from(e.target.files).filter(f => f.type === "audio/mpeg");
      tracks = files;
      trackSelector.innerHTML = "";

      files.forEach((file, i) => {
        const option = document.createElement("option");
        option.value = i;
        option.textContent = file.name;
        trackSelector.appendChild(option);
      });

      if (files.length > 0) {
        playFile(files[0]);
        trackSelector.value = 0;
      }
    });

    function playFile(file) {
      const url = URL.createObjectURL(file);
      audioPlayer.src = url;
      audioPlayer.play();

      mediaTitle.textContent = file.name;
      metaTitle.textContent = ' ';
      metaAlbum.textContent = ' ';
      metaArtist.textContent = ' ';

      jsmediatags.read(file, {
        onSuccess: tag => {
          const tags = tag.tags;
          metaTitle.textContent = tags.title || '(無標題)';
          metaAlbum.textContent = tags.album || '(無專輯)';
          metaArtist.textContent = tags.artist || '(無藝術家)';
          if (tags.title) mediaTitle.textContent = tags.title;
        }
      });
    }

    trackSelector.addEventListener("change", e => {
      playFile(tracks[e.target.value]);
    });

    audioPlayer.addEventListener('ended', () => {
      const idx = parseInt(trackSelector.value);
      if (idx + 1 < tracks.length) {
        trackSelector.value = idx + 1;
        playFile(tracks[idx + 1]);
      }
    });
  </script>
</body>
</html>







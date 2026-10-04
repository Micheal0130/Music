<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mobile Music Player</title>
  <style>
    :root {
      --bg-color: #0f172a;
      --card-bg: rgba(30, 41, 59, 0.8);
      --primary: #38bdf8;
      --primary-hover: #0284c7;
      --text-main: #f8fafc;
      --text-sub: #94a3b8;
      --border: #334155;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    }

    body {
      background-color: var(--bg-color);
      color: var(--text-main);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 16px;
    }

    .player-container {
      width: 100%;
      max-width: 420px;
      background: var(--card-bg);
      backdrop-filter: blur(16px);
      border: 1px solid var(--border);
      border-radius: 24px;
      padding: 24px;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5);
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    .header {
      text-align: center;
    }

    .header h1 {
      font-size: 1.25rem;
      font-weight: 600;
      color: var(--text-main);
    }

    .visualizer-container {
      width: 100%;
      height: 120px;
      background: rgba(15, 23, 42, 0.6);
      border-radius: 16px;
      overflow: hidden;
      display: flex;
      justify-content: center;
      align-items: center;
      border: 1px solid rgba(255, 255, 255, 0.05);
    }

    canvas {
      width: 100%;
      height: 100%;
    }

    .track-info {
      text-align: center;
    }

    .track-title {
      font-size: 1.1rem;
      font-weight: 600;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      margin-bottom: 4px;
    }

    .track-artist {
      font-size: 0.875rem;
      color: var(--text-sub);
    }

    .progress-container {
      display: flex;
      flex-direction: column;
      gap: 6px;
    }

    input[type="range"] {
      -webkit-appearance: none;
      width: 100%;
      height: 6px;
      border-radius: 3px;
      background: #334155;
      outline: none;
      cursor: pointer;
    }

    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: var(--primary);
      cursor: pointer;
      transition: transform 0.1s;
    }

    input[type="range"]::-webkit-slider-thumb:hover {
      transform: scale(1.2);
    }

    .time-stamps {
      display: flex;
      justify-content: space-between;
      font-size: 0.75rem;
      color: var(--text-sub);
    }

    .controls {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0 10px;
    }

    .btn {
      background: none;
      border: none;
      color: var(--text-main);
      font-size: 1.25rem;
      cursor: pointer;
      padding: 8px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.2s, color 0.2s;
    }

    .btn:hover {
      background: rgba(255, 255, 255, 0.1);
      color: var(--primary);
    }

    .btn-play {
      background: var(--primary);
      color: #0f172a;
      width: 52px;
      height: 52px;
      font-size: 1.5rem;
    }

    .btn-play:hover {
      background: var(--primary-hover);
      color: #0f172a;
    }

    .file-input-wrapper {
      text-align: center;
    }

    .file-label {
      display: inline-block;
      padding: 10px 20px;
      background: rgba(56, 189, 248, 0.1);
      color: var(--primary);
      border: 1px solid var(--primary);
      border-radius: 12px;
      font-size: 0.875rem;
      font-weight: 500;
      cursor: pointer;
      transition: all 0.2s;
    }

    .file-label:hover {
      background: var(--primary);
      color: #0f172a;
    }

    input[type="file"] {
      display: none;
    }

    .playlist {
      max-height: 150px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      gap: 6px;
      padding-right: 4px;
    }

    .playlist::-webkit-scrollbar {
      width: 4px;
    }

    .playlist::-webkit-scrollbar-thumb {
      background: var(--border);
      border-radius: 2px;
    }

    .playlist-item {
      padding: 8px 12px;
      background: rgba(15, 23, 42, 0.4);
      border-radius: 8px;
      font-size: 0.85rem;
      cursor: pointer;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      transition: background 0.2s;
    }

    .playlist-item:hover {
      background: rgba(56, 189, 248, 0.15);
    }

    .playlist-item.active {
      background: rgba(56, 189, 248, 0.2);
      color: var(--primary);
      font-weight: 600;
      border-left: 3px solid var(--primary);
    }
  </style>
</head>
<body>

  <div class="player-container">
    <div class="header">
      <h1>My Audio Player</h1>
    </div>

    <div class="visualizer-container">
      <canvas id="visualizer"></canvas>
    </div>

    <div class="track-info">
      <div class="track-title" id="trackTitle">No file loaded</div>
      <div class="track-artist" id="trackArtist">Select music from your device</div>
    </div>

    <div class="progress-container">
      <input type="range" id="progressBar" value="0" min="0" max="100" step="0.1">
      <div class="time-stamps">
        <span id="currentTime">0:00</span>
        <span id="duration">0:00</span>
      </div>
    </div>

    <div class="controls">
      <button class="btn" id="prevBtn" title="Previous">⏮</button>
      <button class="btn btn-play" id="playBtn" title="Play/Pause">▶</button>
      <button class="btn" id="nextBtn" title="Next">⏭</button>
    </div>

    <div class="file-input-wrapper">
      <label for="audioFile" class="file-label">📁 Choose Music Files</label>
      <input type="file" id="audioFile" accept="audio/*" multiple>
    </div>

    <div class="playlist" id="playlist"></div>
  </div>

  <audio id="audio"></audio>

  <script>
    const audio = document.getElementById('audio');
    const playBtn = document.getElementById('playBtn');
    const prevBtn = document.getElementById('prevBtn');
    const nextBtn = document.getElementById('nextBtn');
    const progressBar = document.getElementById('progressBar');
    const currentTimeEl = document.getElementById('currentTime');
    const durationEl = document.getElementById('duration');
    const trackTitle = document.getElementById('trackTitle');
    const trackArtist = document.getElementById('trackArtist');
    const audioFileInput = document.getElementById('audioFile');
    const playlistEl = document.getElementById('playlist');
    const canvas = document.getElementById('visualizer');
    const canvasCtx = canvas.getContext('2d');

    let playlist = [];
    let currentIndex = 0;
    let audioCtx, analyser, source, dataArray;

    // Canvas sizing
    function resizeCanvas() {
      canvas.width = canvas.parentElement.clientWidth;
      canvas.height = canvas.parentElement.clientHeight;
    }
    resizeCanvas();
    window.addEventListener('resize', resizeCanvas);

    // Load Files
    audioFileInput.addEventListener('change', (e) => {
      const files = Array.from(e.target.files);
      if (files.length === 0) return;

      playlist = files.map((file) => ({
        name: file.name.replace(/\.[^/.]+$/, ""),
        url: URL.createObjectURL(file)
      }));

      currentIndex = 0;
      renderPlaylist();
      loadTrack(currentIndex);
      playTrack();
    });

    function renderPlaylist() {
      playlistEl.innerHTML = '';
      playlist.forEach((track, index) => {
        const item = document.createElement('div');
        item.classList.add('playlist-item');
        if (index === currentIndex) item.classList.add('active');
        item.textContent = `${index + 1}. ${track.name}`;
        item.addEventListener('click', () => {
          currentIndex = index;
          loadTrack(currentIndex);
          playTrack();
        });
        playlistEl.appendChild(item);
      });
    }

    function loadTrack(index) {
      if (!playlist[index]) return;
      audio.src = playlist[index].url;
      trackTitle.textContent = playlist[index].name;
      trackArtist.textContent = 'Local Audio';
      renderPlaylist();
    }

    function playTrack() {
      setupAudioContext();
      audio.play();
      playBtn.textContent = '⏸';
      if (audioCtx && audioCtx.state === 'suspended') {
        audioCtx.resume();
      }
      drawVisualizer();
    }

    function pauseTrack() {
      audio.pause();
      playBtn.textContent = '▶';
    }

    playBtn.addEventListener('click', () => {
      if (!audio.src) return;
      if (audio.paused) {
        playTrack();
      } else {
        pauseTrack();
      }
    });

    prevBtn.addEventListener('click', () => {
      if (playlist.length === 0) return;
      currentIndex = (currentIndex - 1 + playlist.length) % playlist.length;
      loadTrack(currentIndex);
      playTrack();
    });

    nextBtn.addEventListener('click', () => {
      if (playlist.length === 0) return;
      currentIndex = (currentIndex + 1) % playlist.length;
      loadTrack(currentIndex);
      playTrack();
    });

    audio.addEventListener('ended', () => {
      nextBtn.click();
    });

    // Progress Bar & Time
    audio.addEventListener('timeupdate', () => {
      if (audio.duration) {
        const progress = (audio.currentTime / audio.duration) * 100;
        progressBar.value = progress;
        currentTimeEl.textContent = formatTime(audio.currentTime);
        durationEl.textContent = formatTime(audio.duration);
      }
    });

    progressBar.addEventListener('input', () => {
      if (audio.duration) {
        audio.currentTime = (progressBar.value / 100) * audio.duration;
      }
    });

    function formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs < 10 ? '0' : ''}${secs}`;
    }

    // Web Audio API Visualizer
    function setupAudioContext() {
      if (audioCtx) return;
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      analyser = audioCtx.createAnalyser();
      source = audioCtx.createMediaElementSource(audio);
      source.connect(analyser);
      analyser.connect(audioCtx.destination);

      analyser.fftSize = 64;
      const bufferLength = analyser.frequencyBinCount;
      dataArray = new Uint8Array(bufferLength);
    }

    function drawVisualizer() {
      if (!analyser) return;

      requestAnimationFrame(drawVisualizer);
      analyser.getByteFrequencyData(dataArray);

      canvasCtx.clearRect(0, 0, canvas.width, canvas.height);

      const barWidth = (canvas.width / dataArray.length) * 1.5;
      let x = 0;

      for (let i = 0; i < dataArray.length; i++) {
        const barHeight = (dataArray[i] / 255) * canvas.height;

        canvasCtx.fillStyle = '#38bdf8';
        canvasCtx.fillRect(x, canvas.height - barHeight, barWidth - 2, barHeight);

        x += barWidth;
      }
    }
  </script>
</body>
</html>

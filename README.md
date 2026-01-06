<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>اپلیکیشن موسیقی من</title>
  <style>
    body {
      font-family: sans-serif;
      background: #0d0d0d;
      color: #fff;
      margin: 0;
      padding: 0;
    }
    header {
      background: #6a0dad;
      padding: 20px;
      text-align: center;
      box-shadow: 0 0 20px #6a0dad;
    }
    h1 { margin: 0; }
    .player-container {
      text-align: center;
      padding: 20px;
    }
    .current-song img {
      width: 250px;
      height: 250px;
      border-radius: 50%;
      object-fit: cover;
      margin-bottom: 10px;
      box-shadow: 0 0 30px #6a0dad;
    }
    .rotate { animation: spin 8s linear infinite; }
    @keyframes spin { from { transform: rotate(0deg);} to { transform: rotate(360deg);} }
    .progress, .volume { width: 100%; margin: 10px 0; }
    input[type="range"] { width: 100%; accent-color: #9b30ff; }
    .time { font-size: 14px; color: #ccc; margin-bottom: 10px; }
    .controls { margin-top: 15px; }
    button {
      background: none; border: none; color: #9b30ff;
      font-size: 28px; margin: 0 15px; cursor: pointer;
      transition: all 0.3s ease;
    }
    button:hover { color: #fff; text-shadow: 0 0 15px #9b30ff; transform: scale(1.2); }
    ul { list-style: none; padding: 0; margin: 20px; }
    li {
      display: flex; align-items: center; justify-content: space-between;
      padding: 10px; border-bottom: 1px solid #333; cursor: pointer;
    }
    li img {
      width: 40px; height: 40px; border-radius: 6px; margin-right: 10px;
      object-fit: cover; box-shadow: 0 0 10px #6a0dad;
    }
    li.active { background: #1a1a1a; font-weight: bold; color: #9b30ff; }
    li:hover { background: #2a2a2a; }
    .add-song { margin: 20px; }
    input { padding: 8px; margin: 5px; border-radius: 6px; border: none; }
    /* ویژوالایزر */
    .visualizer { display: flex; justify-content: center; margin-top: 20px; }
    .bar {
      width: 6px; height: 20px; margin: 0 3px;
      background: #9b30ff; animation: bounce 0.5s infinite alternate;
    }
    @keyframes bounce { from { height: 10px; } to { height: 50px; } }
  </style>
</head>
<body>
  <header><h1>🎶 اپلیکیشن موسیقی من 🎶</h1></header>

  <div class="player-container">
    <div class="current-song">
      <img id="cover" src="cover1.jpg" alt="کاور آهنگ">
      <h2 id="title">آهنگ اول</h2>
      <audio id="player"></audio>
      <div class="progress"><input type="range" id="progress" value="0" min="0" max="100"></div>
      <div class="time"><span id="currentTime">0:00</span> / <span id="duration">0:00</span></div>
      <div class="volume"><label>🔊 ولوم:</label><input type="range" id="volume" value="100" min="0" max="100"></div>
    </div>
    <div class="controls">
      <button id="prev">⏮</button>
      <button id="playPause">▶️</button>
      <button id="next">⏭</button>
    </div>
    <!-- ویژوالایزر -->
    <div class="visualizer">
      <div class="bar"></div><div class="bar"></div><div class="bar"></div>
      <div class="bar"></div><div class="bar"></div><div class="bar"></div>
    </div>
  </div>

  <!-- افزودن آهنگ -->
  <div class="add-song">
    <input type="text" id="songTitle" placeholder="نام آهنگ">
    <input type="text" id="songSrc" placeholder="لینک فایل mp3">
    <input type="text" id="songCover" placeholder="لینک کاور">
    <button id="addBtn">➕ افزودن آهنگ</button>
  </div>

  <ul id="playlist"></ul>

  <script>
    const player = document.getElementById('player');
    const cover = document.getElementById('cover');
    const title = document.getElementById('title');
    const playPauseBtn = document.getElementById('playPause');
    const prevBtn = document.getElementById('prev');
    const nextBtn = document.getElementById('next');
    const playlist = document.getElementById('playlist');
    const progress = document.getElementById('progress');
    const currentTimeEl = document.getElementById('currentTime');
    const durationEl = document.getElementById('duration');
    const volume = document.getElementById('volume');
    const addBtn = document.getElementById('addBtn');
    const songTitle = document.getElementById('songTitle');
    const songSrc = document.getElementById('songSrc');
    const songCover = document.getElementById('songCover');

    let current = 0, isPlaying = false, songs = [];

    // بارگذاری از localStorage
    if (localStorage.getItem("songs")) {
      songs = JSON.parse(localStorage.getItem("songs"));
    } else {
      songs = [
        {title: "آهنگ اول", src: "song1.mp3", cover: "cover1.jpg"},
        {title: "آهنگ دوم", src: "song2.mp3", cover: "cover2.jpg"},
        {title: "آهنگ سوم", src: "song3.mp3", cover: "cover3.jpg"}
      ];
      localStorage.setItem("songs", JSON.stringify(songs));
    }
    renderPlaylist();

    function renderPlaylist() {
      playlist.innerHTML = "";
      songs.forEach((song, index) => {
        const li = document.createElement("li");
        li.setAttribute("data-src", song.src);
        li.setAttribute("data-cover", song.cover);
        li.setAttribute("data-title", song.title);
        li.innerHTML = `<div><img src="${song.cover}" alt="کاور"> ${song.title}</div>
                        <button class="delete">❌ حذف</button>`;
        if (index === current) li.classList.add("active");
        playlist.appendChild(li);
      });
    }

    function playSong(index) {
      current = index;
      const song = songs[index];
      player.src = song.src;
      cover.src = song.cover;
      title.textContent = song.title;
      document.querySelectorAll("#playlist li").forEach(li => li.classList.remove("active"));
      playlist.children[index].classList.add("active");
      player.play();
      isPlaying = true;
      playPauseBtn.textContent = "⏸";
      cover.classList.add("rotate");
    }

    function togglePlayPause() {
      if (isPlaying) {
        player.pause(); isPlaying = false;
        playPauseBtn.textContent = "▶️"; cover.classList.remove("rotate");
<!DOCTYPE html>
<html lang="fa">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>پخش‌کننده موزیک محیا</title>
  <style>
    body {
      font-family: "Tahoma", sans-serif;
      background: linear-gradient(135deg, #1a0033, #4b0082);
      color: #fff;
      text-align: center;
      padding: 40px;
    }
    h1 {
      margin-bottom: 20px;
      animation: fadeIn 2s ease-in-out;
    }
    .player {
      background: rgba(255,255,255,0.1);
      padding: 20px;
      border-radius: 12px;
      display: inline-block;
    }
    button {
      margin: 10px;
      padding: 10px 20px;
      border: none;
      border-radius: 8px;
      background: #6a0dad;
      color: #fff;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover {
      background: #9b30ff;
    }
    @keyframes fadeIn {
      from {opacity: 0;}
      to {opacity: 1;}
    }
  </style>
</head>
<body>
  <h1>🎵 پخش‌کننده موزیک محیا 🎵</h1>
  
  <div class="player">
    <audio id="audio" src="music.mp3"></audio>
    <button onclick="playMusic()">▶️ پخش</button>
    <button onclick="pauseMusic()">⏸ توقف</button>
    <button onclick="stopMusic()">⏹ پایان</button>
  </div>

  <script>
    const audio = document.getElementById("audio");

    function playMusic() {
      audio.play();
    }
    function pauseMusic() {
      audio.pause();
    }
    function stopMusic() {
      audio.pause();
      audio.currentTime = 0;
    }
  </script>
</body>
</html>
<link rel="stylesheet" href="./assets/style.css">
<script src="./assets/script.js"></script>

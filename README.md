<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Twitch Онлайн - Браузер</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    /* Стили те же самые, что и раньше */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #18181b;
      color: white;
      display: flex;
      flex-direction: column;
      min-height: 100vh;
    }

    header {
      background-color: #0e0e10;
      padding: 15px 20px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .logo {
      font-size: 24px;
      font-weight: bold;
      color: #9147ff;
    }
    nav a {
      color: white;
      margin-left: 20px;
      text-decoration: none;
      transition: color 0.3s;
    }
    nav a:hover {
      color: #9147ff;
    }

    main {
      display: flex;
      flex: 1;
      padding: 20px;
      gap: 20px;
      flex-wrap: wrap;
    }

    aside {
      width: 250px;
      background-color: #27272a;
      border-radius: 8px;
      padding: 15px;
      flex-shrink: 0;
    }
    .search-input {
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 4px;
      margin-bottom: 15px;
      font-size: 16px;
    }
    .streamer-list {
      list-style: none;
    }
    .streamer {
      padding: 10px;
      background-color: #1f1f22;
      border-radius: 6px;
      cursor: pointer;
      margin-bottom: 10px;
      transition: background-color 0.3s;
    }
    .streamer:hover {
      background-color: #333;
    }

    .stream-container {
      flex: 1;
      display: flex;
      flex-direction: column;
      gap: 20px;
    }
    .stream-player {
      width: 100%;
      height: 500px;
      border: none;
      border-radius: 8px;
      background-color: black;
    }
    .chat {
      flex: 1;
      width: 100%;
      height: 500px;
      border: none;
      border-radius: 8px;
      background-color: #0e0e10;
    }

    .favorites {
      margin-top: 20px;
      padding-top: 10px;
      border-top: 1px solid #333;
    }

    @media (max-width: 768px) {
      main {
        flex-direction: column;
      }
      aside {
        width: 100%;
      }
      .stream-container {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>

  <!-- Шапка -->
  <header>
    <div class="logo">Twitch Online</div>
    <nav>
      <a href="#">Главная</a>
      <a href="#">Категории</a>
      <a href="#">Чат</a>
      <a href="#" id="authLink">Вход</a>
    </nav>
  </header>

  <!-- Основной контент -->
  <main>
    <!-- Левое меню -->
    <aside>
      <input type="text" class="search-input" placeholder="Поиск стримера..." onkeyup="filterStreamers()">
      <ul class="streamer-list" id="streamerList"></ul>

      <div class="favorites" id="favoritesSection">
        <h4>⭐ Избранное</h4>
        <ul class="streamer-list" id="favoritesList"></ul>
      </div>
    </aside>

    <!-- Плеер и чат -->
    <div class="stream-container">
      <iframe class="stream-player" id="player" src="" allowfullscreen></iframe>
      <iframe class="chat" id="chat" src="https://www.twitch.tv/embed/xqc/chat?parent=localhost " frameborder="0"></iframe>
    </div>
  </main>

  <!-- JS -->
  <script>
    const clientId = 'kimne78kx3ncx0'; // Общий Client ID
    const redirectUri = encodeURIComponent('http://localhost'); // или свой домен
    let accessToken = localStorage.getItem('twitch_token');
    let favorites = JSON.parse(localStorage.getItem('favorites') || '[]');

    document.getElementById('authLink').innerText = accessToken ? 'Выход' : 'Вход';
    document.getElementById('authLink').onclick = () => {
      if (accessToken) {
        localStorage.removeItem('twitch_token');
        accessToken = null;
        document.getElementById('authLink').innerText = 'Вход';
      } else {
        window.location.href = `https://id.twitch.tv/oauth2/authorize?response_type=token&client_id= ${clientId}&redirect_uri=${redirectUri}&scope=user:read:email`;
      }
    };

    async function fetchOnlineStreamers() {
      const res = await fetch('https://api.twitch.tv/helix/streams?first=20 ', {
        headers: { 'Client-ID': clientId }
      });
      const data = await res.json();
      const streams = data.data;

      const userRes = await fetch(`https://api.twitch.tv/helix/users? ${streams.map(s => `id=${s.user_id}`).join('&')}`, {
        headers: { 'Client-ID': clientId }
      });
      const usersData = await userRes.json();

      const streamers = streams.map(stream => {
        const user = usersData.data.find(u => u.id === stream.user_id);
        return {
          name: stream.user_login,
          title: stream.title,
          viewer_count: stream.viewer_count,
          game: stream.game_name
        };
      });

      renderStreamers(streamers);
    }

    function renderStreamers(streamers) {
      const container = document.getElementById('streamerList');
      container.innerHTML = '';
      streamers.forEach(streamer => {
        const li = document.createElement('li');
        li.className = 'streamer';
        li.innerText = `${streamer.name} (${streamer.viewer_count})`;
        li.onclick = () => loadStream(streamer.name);
        container.appendChild(li);
      });
    }

    function filterStreamers() {
      const term = document.querySelector('.search-input').value.toLowerCase();
      const streamers = Array.from(document.querySelectorAll('.streamer'));
      streamers.forEach(s => {
        const name = s.innerText.toLowerCase();
        s.style.display = name.includes(term) ? 'block' : 'none';
      });
    }

    function loadStream(channel) {
      document.getElementById('player').src = `https://player.twitch.tv/?channel= ${channel}&parent=localhost`;
      document.getElementById('chat').src = `https://www.twitch.tv/embed/ ${channel}/chat?parent=localhost`;

      if (!favorites.includes(channel)) {
        const confirmSub = confirm(`Добавить ${channel} в избранное?`);
        if (confirmSub) {
          favorites.push(channel);
          localStorage.setItem('favorites', JSON.stringify(favorites));
          renderFavorites();
        }
      }
    }

    function renderFavorites() {
      const container = document.getElementById('favoritesList');
      container.innerHTML = '';
      favorites.forEach(name => {
        const li = document.createElement('li');
        li.className = 'streamer';
        li.innerText = name;
        li.onclick = () => loadStream(name);
        container.appendChild(li);
      });
    }

    fetchOnlineStreamers();
    renderFavorites();
  </script>

</body>
</html>

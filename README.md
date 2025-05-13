<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Twitch Онлайн - Браузер</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background-color: #18181b;
      color: white;
    }

    header {
      background-color: #0e0e10;
      padding: 15px 20px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .logo {
      font-size: 20px;
      font-weight: bold;
      color: #9147ff;
    }

    nav a {
      color: white;
      margin-left: 15px;
      text-decoration: none;
    }

    main {
      display: flex;
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

    input[type="text"] {
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 4px;
      margin-bottom: 15px;
    }

    ul {
      list-style: none;
    }

    li {
      padding: 10px;
      background-color: #1f1f22;
      border-radius: 6px;
      cursor: pointer;
      margin-bottom: 10px;
      transition: background-color 0.3s;
    }

    li:hover {
      background-color: #333;
    }

    .stream-container {
      flex: 1;
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    iframe {
      width: 100%;
      height: 500px;
      border: none;
      border-radius: 8px;
    }

    .favorites {
      margin-top: 20px;
      padding-top: 10px;
      border-top: 1px solid #333;
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
    </nav>
  </header>

  <!-- Основной контент -->
  <main>
    <!-- Левое меню -->
    <aside>
      <input type="text" placeholder="Поиск стримера..." onkeyup="filterStreamers()">
      <ul id="streamerList">
        <li onclick="loadStream('xqc')">xqc</li>
        <li onclick="loadStream('asmongold')">asmongold</li>
        <li onclick="loadStream('shroud')">shroud</li>
        <li onclick="loadStream('summit1g')">summit1g</li>
        <li onclick="loadStream('pokimane')">pokimane</li>
        <li onclick="loadStream('timthetatman')">timthetatman</li>
        <li onclick="loadStream('monstercat')">monstercat</li>
      </ul>

      <div class="favorites">
        <h4>⭐ Избранное</h4>
        <ul id="favoritesList"></ul>
      </div>
    </aside>

    <!-- Трансляция -->
    <div class="stream-container">
      <iframe id="player" src="" allowfullscreen></iframe>
      <iframe id="chat" src=""></iframe>
    </div>
  </main>

  <!-- JS -->
  <script>
    const favorites = JSON.parse(localStorage.getItem('favorites') || '[]');
    const favoritesList = document.getElementById('favoritesList');

    function loadStream(channel) {
      document.getElementById('player').src = `https://player.twitch.tv/?channel= ${channel}&parent=localhost`;
      document.getElementById('chat').src = `https://www.twitch.tv/embed/ ${channel}/chat?parent=localhost`;

      if (!favorites.includes(channel)) {
        if (confirm(`Добавить ${channel} в избранное?`)) {
          favorites.push(channel);
          localStorage.setItem('favorites', JSON.stringify(favorites));
          renderFavorites();
        }
      }
    }

    function renderFavorites() {
      favoritesList.innerHTML = '';
      favorites.forEach(name => {
        const li = document.createElement('li');
        li.textContent = name;
        li.onclick = () => loadStream(name);
        favoritesList.appendChild(li);
      });
    }

    function filterStreamers() {
      const term = document.querySelector('input').value.toLowerCase();
      const streamers = document.querySelectorAll('#streamerList li');
      streamers.forEach(s => {
        const name = s.textContent.toLowerCase();
        s.style.display = name.includes(term) ? 'block' : 'none';
      });
    }

    renderFavorites();
  </script>

</body>
</html>

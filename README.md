<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Player Lista M3U</title>
  <style>
    body { font-family: sans-serif; margin:0; display:flex; height:100vh; background:#0b0b0d; color:#fff; }
    .left { width:280px; padding:16px; background:#111; overflow-y:auto; }
    .player { flex:1; display:flex; flex-direction:column; align-items:center; justify-content:flex-start; padding:16px; }
    .media { width:100%; max-width:960px; height:540px; background:#000; border-radius:8px; overflow:hidden; }
    video, iframe { width:100%; height:100%; border:0; }
    h1 { font-size:18px; margin:0 0 12px; }
    ul { list-style:none; padding:0; margin:0; }
    li { padding:10px; margin-bottom:8px; border-radius:6px; cursor:pointer; background:#222; }
    li:hover { background:#333; }
    #now { margin-top:10px; opacity:0.8; }
  </style>
</head>
<body>
  <div class="left">
    <h1>Canais</h1>
    <ul id="channels"></ul>
  </div>

  <div class="player">
    <div class="media" id="media"></div>
    <div id="now">Selecione um canal</div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
  <script>
    async function loadPlaylist() {
      const res = await fetch('playlist.m3u');
      const text = await res.text();
      const lines = text.split(/\r?\n/).map(l => l.trim()).filter(Boolean);
      const channels = [];
      for (let i=0; i<lines.length; i++) {
        if (lines[i].startsWith('#EXTINF')) {
          const name = lines[i].split(',').slice(1).join(',').trim();
          const url = lines[i+1] || '';
          channels.push({ name, url });
        }
      }
      renderChannels(channels);
    }

    function renderChannels(channels) {
      const ul = document.getElementById('channels');
      ul.innerHTML = '';
      channels.forEach(ch => {
        const li = document.createElement('li');
        li.textContent = ch.name;
        li.onclick = () => playChannel(ch);
        ul.appendChild(li);
      });
    }

    function playChannel(ch) {
      const container = document.getElementById('media');
      container.innerHTML = '';
      document.getElementById('now').textContent = 'Tocando: ' + ch.name;

      if (ch.url.startsWith('embed::')) {
        const iframe = document.createElement('iframe');
        iframe.src = ch.url.replace('embed::','');
        iframe.allowFullscreen = true;
        container.appendChild(iframe);
      } else {
        const video = document.createElement('video');
        video.controls = true;
        container.appendChild(video);

        if (Hls.isSupported()) {
          if (window.hls) window.hls.destroy();
          const hls = new Hls();
          hls.loadSource(ch.url);
          hls.attachMedia(video);
          window.hls = hls;
        } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
          video.src = ch.url;
        } else {
          container.innerHTML = '<p>Navegador não suporta HLS</p>';
        }
        video.play().catch(()=>{});
      }
    }

    loadPlaylist();
  </script>
</body>
</html>

# prova1
<!DOCTYPE html>
<html lang="ca">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Meteorit Launcher Map</title>
  //unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    html, body {
      margin: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
      background: #0b1020;
    }
    #app {
      position: relative;
      width: 100vw;
      height: 100vh;
    }
    #map {
      width: 100%;
      height: 100%;
    }
    .panel {
      position: absolute;
      top: 16px;
      left: 16px;
      z-index: 1000;
      width: 320px;
      padding: 16px;
      border-radius: 16px;
      background: rgba(10, 14, 28, 0.88);
      color: white;
      box-shadow: 0 12px 40px rgba(0,0,0,0.35);
      backdrop-filter: blur(10px);
    }
    .panel h1 {
      font-size: 18px;
      margin: 0 0 12px;
    }
    .row {
      margin-bottom: 12px;
    }
    label {
      display: block;
      margin-bottom: 4px;
      font-size: 13px;
      opacity: 0.9;
    }
    input, select, button {
      width: 100%;
      box-sizing: border-box;
      border: none;
      border-radius: 10px;
      padding: 10px 12px;
      font-size: 14px;
    }
    input, select {
      background: #151c33;
      color: white;
      outline: none;
    }
    button {
      background: linear-gradient(135deg, #ff5f6d, #ff9966);
      color: white;
      cursor: pointer;
      font-weight: 700;
    }
    button:hover {
      filter: brightness(1.05);
    }
    .stats {
      margin-top: 10px;
      font-size: 13px;
      line-height: 1.5;
      color: #d7def5;
    }
    .hint {
      font-size: 12px;
      opacity: 0.75;
      margin-top: 8px;
    }
    .crosshair {
      width: 24px;
      height: 24px;
      border: 2px solid #ffcc00;
      border-radius: 50%;
      box-shadow: 0 0 12px rgba(255, 204, 0, 0.7);
      transform: translate(-50%, -50%);
    }
    .impact-marker {
      width: 18px;
      height: 18px;
      border-radius: 50%;
      background: #ff3b30;
      box-shadow: 0 0 0 0 rgba(255,59,48,0.7);
      animation: pulse 1.5s infinite;
      transform: translate(-50%, -50%);
    }
    @keyframes pulse {
      0% { box-shadow: 0 0 0 0 rgba(255,59,48,0.65); }
      70% { box-shadow: 0 0 0 22px rgba(255,59,48,0); }
      100% { box-shadow: 0 0 0 0 rgba(255,59,48,0); }
    }
    .trail {
      width: 4px;
      height: 4px;
      border-radius: 50%;
      background: #ffd54a;
      box-shadow: 0 0 10px #ffd54a;
      transform: translate(-50%, -50%);
    }
  </style>
</head>
<body>
  <div id="app">
    <div id="map"></div>

    <div class="panel">
      <h1>Meteorit Launcher</h1>

      <div class="row">
        abel for="size">Mida del meteorit</label>
        <input id="size" type="range" min="10" max="1000" value="250" />
      </div>

      <div class="row">
        abel for="speed">Velocitat</label>
        <input id="speed" type="range" min="5" max="70" value="20" />
      </div>

      <div class="row">
        abel for="angle">Angle d’impacte</label>
        <input id="angle" type="range" min="10" max="90" value="45" />
      </div>

      <div class="row">
        abel for="material">Material</label>
        <select id="material">
          <option value="stone">Pedra</option>
          <option value="iron">Ferro</option>
          <option value="ice">Gel</option>
        </select>
      </div>

      <div class="row">
        <button id="launch">Llança el meteorit</button>
      </div>

      <div class="stats" id="stats">
        Fes clic al mapa per triar el punt d’impacte.
      </div>
      <div class="hint">
        Inspirat en un simulador tipus mapa i en l’estil d’Asteroid Launcher.
      </div>
    </div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const map = L.map('map', { zoomControl: true }).setView([41.3879, 2.16992], 5);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map);

    let target = null;
    let targetMarker = null;
    let impactCircle = null;
    let trailLine = null;
    let trailMarker = null;

    const stats = document.getElementById('stats');
    const sizeEl = document.getElementById('size');
    const speedEl = document.getElementById('speed');
    const angleEl = document.getElementById('angle');
    const materialEl = document.getElementById('material');

    function materialFactor(m) {
      if (m === 'iron') return 1.25;
      if (m === 'ice') return 0.7;
      return 1.0;
    }

    function clearEffects() {
      if (impactCircle) map.removeLayer(impactCircle);
      if (trailLine) map.removeLayer(trailLine);
      if (trailMarker) map.removeLayer(trailMarker);
      impactCircle = null;
      trailLine = null;
      trailMarker = null;
    }

    map.on('click', (e) => {
      target = e.latlng;
      if (targetMarker) map.removeLayer(targetMarker);

      targetMarker = L.marker(target, {
        icon: L.divIcon({
          className: '',
          html: '<div class="crosshair"></div>',
          iconSize: [24, 24],
          iconAnchor: [12, 12]
        })
      }).addTo(map);

      stats.innerHTML = `
        Punt seleccionat: ${target.lat.toFixed(4)}, ${target.lng.toFixed(4)}<br>
        Prem “Llança el meteorit” per impactar.
      `;
    });

    document.getElementById('launch').addEventListener('click', () => {
      if (!target) {
        stats.innerHTML = 'Primer fes clic al mapa per marcar un punt.';
        return;
      }

      clearEffects();

      const size = parseFloat(sizeEl.value);
      const speed = parseFloat(speedEl.value);
      const angle = parseFloat(angleEl.value);
      const material = materialEl.value;
      const factor = materialFactor(material);

      const craterKm = (size * factor * (speed / 20) * (Math.sin(angle * Math.PI / 180) + 0.3)) / 10;
      const blastKm = craterKm * 3.2;

      impactCircle = L.circle(target, {
        radius: blastKm * 1000,
        color: '#ff6b00',
        fillColor: '#ff3b30',
        fillOpacity: 0.18,
        weight: 2
      }).addTo(map);

      const start = map.getBounds().getNorthWest();
      const launchPoint = L.latLng(start.lat, target.lng - 5);

      trailLine = L.polyline([launchPoint, target], {
        color: '#ffd54a',
        weight: 3,
        opacity: 0.9
      }).addTo(map);

      trailMarker = L.marker(launchPoint, {
        icon: L.divIcon({
          className: '',
          html: '<div class="trail"></div>',
          iconSize: [4, 4],
          iconAnchor: [2, 2]
        })
      }).addTo(map);

      let steps = 0;
      const totalSteps = 80;
      const interval = setInterval(() => {
        steps++;
        const t = steps / totalSteps;
        const lat = launchPoint.lat + (target.lat - launchPoint.lat) * t;
        const lng = launchPoint.lng + (target.lng - launchPoint.lng) * t;
        trailMarker.setLatLng([lat, lng]);

        if (steps >= totalSteps) {
          clearInterval(interval);
          trailMarker.remove();

          L.marker(target, {
            icon: L.divIcon({
              className: '',
              html: '<div class="impact-marker"></div>',
              iconSize: [18, 18],
              iconAnchor: [9, 9]
            })
          }).addTo(map);

          stats.innerHTML = `
            <b>Impacte registrat</b><br>
            Material: ${material}<br>
            Mida: ${size} m<br>
            Velocitat: ${speed} km/s<br>
            Angle: ${angle}°<br>
            Cràter estimat: ${craterKm.toFixed(2)} km<br>
            Zona afectada: ${blastKm.toFixed(2)} km de radi
          `;
        }
      }, 20);
    });
  </script>
</body>
</html>

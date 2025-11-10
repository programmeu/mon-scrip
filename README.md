<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Météo Précise – Ville ou Code Postal</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(to bottom, #b2ebf2, #e0f7fa);
      padding: 20px;
      margin: 0;
    }
    .container {
      max-width: 700px;
      margin: auto;
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
      color: #00796b;
    }
    input, select, button, textarea {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      font-size: 1em;
    }
    table {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: center;
      font-size: 0.95em;
    }
    th {
      background-color: #b2dfdb;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🌦️ Météo Précise</h1>

    <label for="ville">Choisis une ville :</label>
    <select id="ville">
      <option value="">-- Aucune --</option>
      <option value="Mazamet">Mazamet</option>
      <option value="Castres">Castres</option>
      <option value="Aussillon">Aussillon</option>
      <option value="Toulouse">Toulouse</option>
      <option value="Albi">Albi</option>
      <option value="Paris">Paris</option>
    </select>

    <label for="codePostal">Ou entre un code postal :</label>
    <input type="text" id="codePostal" placeholder="Ex: 81200">

    <label for="duree">Durée des prévisions :</label>
    <select id="duree">
      <option value="1">Aujourd’hui</option>
      <option value="3">3 jours</option>
      <option value="7">1 semaine</option>
    </select>

    <label for="domaine">Domaine météo :</label>
    <select id="domaine">
      <option value="tout">Tout</option>
      <option value="temperature">Température</option>
      <option value="pluie">Pluie</option>
      <option value="vent">Vent</option>
    </select>

    <button onclick="getMeteo()">Voir la météo</button>
    <div id="resultat"></div>

    <h2>📝 Tes notes météo</h2>
    <textarea id="notes" rows="5" placeholder="Écris ici tes observations..."></textarea>
    <button onclick="sauverNotes()">💾 Sauvegarder</button>
  </div>

  <script>
    const villes = {
      "Mazamet": { lat: 43.4922, lon: 2.3736 },
      "Castres": { lat: 43.6043, lon: 2.2416 },
      "Aussillon": { lat: 43.4975, lon: 2.3747 },
      "Toulouse": { lat: 43.6047, lon: 1.4442 },
      "Albi": { lat: 43.9298, lon: 2.148 },
      "Paris": { lat: 48.8566, lon: 2.3522 }
    };

    async function getMeteo() {
      const ville = document.getElementById("ville").value;
      const codePostal = document.getElementById("codePostal").value.trim();
      const nbJours = parseInt(document.getElementById("duree").value);
      const domaine = document.getElementById("domaine").value;

      let lat, lon, nomLieu;

      if (ville) {
        ({ lat, lon } = villes[ville]);
        nomLieu = ville;
      } else if (codePostal) {
        const geoUrl = `https://api.zippopotam.us/fr/${codePostal}`;
        const geoRes = await fetch(geoUrl);
        if (!geoRes.ok) {
          alert("Code postal invalide ou non trouvé.");
          return;
        }
        const geoData = await geoRes.json();
        const place = geoData.places[0];
        lat = place.latitude;
        lon = place.longitude;
        nomLieu = place["place name"];
      } else {
        alert("Choisis une ville ou entre un code postal.");
        return;
      }

      const meteoUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=temperature_2m_max,temperature_2m_min,precipitation_sum,windspeed_10m_max&timezone=auto`;
      const meteoRes = await fetch(meteoUrl);
      const data = await meteoRes.json();

      const jours = data.daily.time.slice(0, nbJours);
      const tempMax = data.daily.temperature_2m_max.slice(0, nbJours);
      const tempMin = data.daily.temperature_2m_min.slice(0, nbJours);
      const pluie = data.daily.precipitation_sum.slice(0, nbJours);
      const vent = data.daily.windspeed_10m_max.slice(0, nbJours);

      let html = `<h2>Prévisions pour ${nomLieu} – ${nbJours} jour(s)</h2><table><tr><th>Date</th>`;
      if (domaine === "temperature" || domaine === "tout") {
        html += `<th>🌡️ Min (°C)</th><th>🌡️ Max (°C)</th>`;
      }
      if (domaine === "pluie" || domaine === "tout") {
        html += `<th>🌧️ Pluie (mm)</th>`;
      }
      if (domaine === "vent" || domaine === "tout") {
        html += `<th>🌬️ Vent (km/h)</th>`;
      }
      html += `</tr>`;

      for (let i = 0; i < jours.length; i++) {
        html += `<tr><td>${jours[i]}</td>`;
        if (domaine === "temperature" || domaine === "tout") {
          html += `<td>${tempMin[i]}</td><td>${tempMax[i]}</td>`;
        }
        if (domaine === "pluie" || domaine === "tout") {
          html += `<td>${pluie[i]}</td>`;
        }
        if (domaine === "vent" || domaine === "tout") {
          html += `<td>${vent[i]}</td>`;
        }
        html += `</tr>`;
      }
      html += `</table>`;
      document.getElementById("resultat").innerHTML = html;

      chargerNotes(nomLieu);
    }

    function sauverNotes() {
      const ville = document.getElementById("ville").value;
      const codePostal = document.getElementById("codePostal").value.trim();
      const cle = ville || codePostal;
      const texte = document.getElementById("notes").value;
      localStorage.setItem("notes_" + cle, texte);
      alert("Tes notes pour " + cle + " ont été enregistrées !");
    }

    function chargerNotes(cle) {
      const texte = localStorage.getItem("notes_" + cle) || "";
      document.getElementById("notes").value = texte;
    }

    // Recharge automatique après minuit
    setInterval(() => {
      const maintenant = new Date();
      if (maintenant.getHours() === 0 && maintenant.getMinutes() === 0) {
        getMeteo();
      }
    }, 60000);
  </script>
</body>
</html>

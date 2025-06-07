<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weather App</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Orbitron&display=swap">
  <style>
    :root {
      --light-bg: linear-gradient(to right, #a1c4fd, #c2e9fb);
      --dark-bg: linear-gradient(to right, #232526, #414345);
      --text-light: #000;
      --text-dark: #fff;
      --container-bg-light: rgba(255, 255, 255, 0.3);
      --container-bg-dark: rgba(0, 0, 0, 0.5);
    }

    body {
      margin: 0;
      font-family: 'Orbitron', sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      transition: all 0.4s ease-in-out;
      background: var(--light-bg);
      color: var(--text-light);
    }

    body.dark {
      background: var(--dark-bg);
      color: var(--text-dark);
    }

    .container {
      text-align: center;
      padding: 30px;
      border-radius: 15px;
      backdrop-filter: blur(10px);
      transition: 0.4s;
      background: var(--container-bg-light);
    }

    body.dark .container {
      background: var(--container-bg-dark);
    }

    input, button {
      padding: 10px;
      border-radius: 5px;
      border: none;
      margin: 5px;
      font-size: 16px;
    }

    button {
      background-color: #ff4b2b;
      color: white;
      cursor: pointer;
    }

    button:hover {
      background-color: #ff6b4b;
    }

    .weather-box, .forecast-box {
      margin-top: 20px;
      font-size: 18px;
    }

    .forecast-day {
      margin-top: 10px;
    }

    .toggle-btn {
      position: absolute;
      top: 10px;
      right: 10px;
      padding: 8px 14px;
      background: #333;
      color: #fff;
      border-radius: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="toggle-btn" onclick="toggleTheme()">Toggle Dark/Light</div>
  <div class="container">
    <h1>🌤 Weather App</h1>
    <input type="text" id="cityInput" placeholder="Enter city name">
    <button onclick="getWeatherByCity()">Get Weather</button>
    <button onclick="getWeatherByLocation()">📍 Use My Location</button>

    <div id="weatherInfo" class="weather-box"></div>
    <div id="forecastInfo" class="forecast-box"></div>
  </div>

  <script>
    const apiKey = "59aab28ca94d2ea6ead234e9fe1ddee3"; 
    function toggleTheme() {
      document.body.classList.toggle('dark');
    }

    function setBackground(weather) {
      const body = document.body;
      if (weather.includes("cloud")) body.style.background = "linear-gradient(to right, #757f9a, #d7dde8)";
      else if (weather.includes("rain")) body.style.background = "linear-gradient(to right, #485563, #29323c)";
      else if (weather.includes("clear")) body.style.background = "linear-gradient(to right, #56ccf2, #2f80ed)";
      else if (weather.includes("snow")) body.style.background = "linear-gradient(to right, #e6dada, #274046)";
      else body.style.background = "linear-gradient(to right, #a1c4fd, #c2e9fb)";
    }

    function getWeatherByCity() {
      const city = document.getElementById("cityInput").value;
      if (city) fetchWeather(city);
    }

    function getWeatherByLocation() {
      navigator.geolocation.getCurrentPosition(pos => {
        const { latitude, longitude } = pos.coords;
        fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apiKey}&units=metric`)
          .then(res => res.json())
          .then(data => {
            showWeather(data);
            fetchForecast(data.name);
          });
      });
    }

    function fetchWeather(city) {
      fetch(`https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${apiKey}&units=metric`)
        .then(res => res.json())
        .then(data => {
          showWeather(data);
          fetchForecast(city);
        });
    }

    function showWeather(data) {
      const weatherBox = document.getElementById("weatherInfo");
      const { name } = data;
      const { temp, humidity } = data.main;
      const { speed } = data.wind;
      const { icon, description } = data.weather[0];

      setBackground(description.toLowerCase());

      weatherBox.innerHTML = `
        <h2>📍 ${name}</h2>
        <img src="https://openweathermap.org/img/wn/${icon}@2x.png" alt="${description}">
        <p><strong>Temperature:</strong> ${temp}°C</p>
        <p><strong>Condition:</strong> ${description}</p>
        <p><strong>Humidity:</strong> ${humidity}%</p>
        <p><strong>Wind Speed:</strong> ${speed} m/s</p>
      `;
    }

    function fetchForecast(city) {
      fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`)
        .then(res => res.json())
        .then(data => {
          const forecastBox = document.getElementById("forecastInfo");
          const list = data.list;
          forecastBox.innerHTML = '<h3>📆 5-Day Forecast</h3>';
          for (let i = 0; i < list.length; i += 8) {
            const day = list[i];
            const date = new Date(day.dt_txt).toLocaleDateString();
            forecastBox.innerHTML += `
              <div class="forecast-day">
                <strong>${date}</strong><br>
                <img src="https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png" alt="${day.weather[0].description}">
                <p>${day.weather[0].description}, ${day.main.temp}°C</p>
              </div>
            `;
          }
        });
    }
  </script>
</body>
</html>

# Asynchronous-JavaScript-RESTful-APIs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-Time Weather Dashboard</title>
    <style>
        :root {
            --primary-color: #2563eb;
            --bg-color: #f8fafc;
            --card-bg: #ffffff;
            --text-main: #1e293b;
            --text-muted: #64748b;
            --error: #ef4444;
        }

        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }

        .dashboard {
            background-color: var(--card-bg);
            padding: 2.5rem;
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 450px;
        }

        h1 {
            margin-top: 0;
            font-size: 1.5rem;
            color: var(--primary-color);
            margin-bottom: 1.5rem;
            text-align: center;
        }

        .search-box {
            display: flex;
            gap: 10px;
            margin-bottom: 1.5rem;
        }

        input {
            flex: 1;
            padding: 12px;
            border: 1px solid #cbd5e1;
            border-radius: 6px;
            font-size: 1rem;
            outline: none;
            transition: border-color 0.2s;
        }

        input:focus {
            border-color: var(--primary-color);
        }

        button {
            padding: 12px 20px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 1rem;
            cursor: pointer;
            font-weight: 600;
            transition: background-color 0.2s;
        }

        button:hover {
            background-color: #1d4ed8;
        }

        .error-message {
            color: var(--error);
            background-color: #fef2f2;
            border: 1px solid #fca5a5;
            padding: 10px;
            border-radius: 6px;
            margin-bottom: 1rem;
            display: none;
            text-align: center;
        }

        .weather-results {
            display: none;
            flex-direction: column;
            gap: 15px;
            border-top: 1px solid #e2e8f0;
            padding-top: 1.5rem;
        }

        .city-title {
            font-size: 1.75rem;
            margin: 0;
            text-align: center;
        }

        .metric-card {
            background-color: #f1f5f9;
            padding: 15px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .metric-label {
            color: var(--text-muted);
            font-weight: 600;
        }

        .metric-value {
            font-size: 1.25rem;
            font-weight: bold;
            color: var(--text-main);
        }

        /* Utility classes for toggling visibility */
        .show { display: block; }
        .show-flex { display: flex; }
    </style>
</head>
<body>

    <div class="dashboard">
        <h1>Weather Dashboard</h1>
        
        <div class="search-box">
            <input type="text" id="cityInput" placeholder="Enter city name (e.g., London)">
            <button id="searchBtn">Search</button>
        </div>

        <div id="errorContainer" class="error-message"></div>

        <div id="weatherContainer" class="weather-results">
            <h2 id="cityName" class="city-title"></h2>
            
            <div class="metric-card">
                <span class="metric-label">Temperature</span>
                <span class="metric-value"><span id="tempValue">--</span>°C</span>
            </div>
            
            <div class="metric-card">
                <span class="metric-label">Humidity</span>
                <span class="metric-value"><span id="humidityValue">--</span>%</span>
            </div>
            
            <div class="metric-card">
                <span class="metric-label">Wind Speed</span>
                <span class="metric-value"><span id="windValue">--</span> km/h</span>
            </div>
        </div>
    </div>

    <script>
        document.getElementById('searchBtn').addEventListener('click', fetchWeatherData);
        
        // Allow user to hit "Enter" in the input field
        document.getElementById('cityInput').addEventListener('keypress', function (e) {
            if (e.key === 'Enter') {
                fetchWeatherData();
            }
        });

        async function fetchWeatherData() {
            const cityInput = document.getElementById('cityInput').value.trim();
            const weatherContainer = document.getElementById('weatherContainer');
            const errorContainer = document.getElementById('errorContainer');

            // Reset UI states
            weatherContainer.classList.remove('show-flex');
            errorContainer.classList.remove('show');
            errorContainer.textContent = '';

            if (!cityInput) {
                showError("Please enter a city name.");
                return;
            }

            try {
                // Step 1: Geocoding API to get Latitude & Longitude from city name
                const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(cityInput)}&count=1&language=en&format=json`;
                const geoResponse = await fetch(geoUrl);
                
                if (!geoResponse.ok) {
                    throw new Error("Failed to connect to the geolocation service.");
                }

                const geoData = await geoResponse.json();

                // Check if the city was found
                if (!geoData.results || geoData.results.length === 0) {
                    throw new Error("City not found. Please check your spelling.");
                }

                const location = geoData.results[0];

                // Step 2: Fetch Weather Data using coordinates (Parsing nested JSON)
                const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${location.latitude}&longitude=${location.longitude}&current=temperature_2m,relative_humidity_2m,wind_speed_10m`;
                const weatherResponse = await fetch(weatherUrl);

                if (!weatherResponse.ok) {
                    throw new Error("Failed to retrieve weather data.");
                }

                const weatherData = await weatherResponse.json();
                
                // Step 3: Parse nested JSON and Update the UI dynamically
                document.getElementById('cityName').textContent = location.name;
                document.getElementById('tempValue').textContent = weatherData.current.temperature_2m;
                document.getElementById('humidityValue').textContent = weatherData.current.relative_humidity_2m;
                document.getElementById('windValue').textContent = weatherData.current.wind_speed_10m;

                // Display the results
                weatherContainer.classList.add('show-flex');

            } catch (error) {
                // Comprehensive error handling
                console.error("Fetch Error:", error);
                showError(error.message || "An unexpected error occurred. Please try again.");
            }
        }

        function showError(message) {
            const errorContainer = document.getElementById('errorContainer');
            errorContainer.textContent = message;
            errorContainer.classList.add('show');
        }
    </script>
</body>
</html>
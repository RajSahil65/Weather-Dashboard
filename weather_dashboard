import streamlit as st
import requests
import datetime
import pytz
from timezonefinder import TimezoneFinder
import plotly.graph_objects as go

# --- Config ---
API_KEY = "3fa4bbed1ca12d50395d3ce29070628a"
BASE_URL = "http://api.openweathermap.org/data/2.5/weather"
FORECAST_URL = "http://api.openweathermap.org/data/2.5/forecast"
FLAG_URL = "https://flagcdn.com/80x60/{code}.png"

# --- Streamlit Page Setup ---
st.set_page_config(page_title="🌍 Global Weather", layout="centered")
st.title("🌍 Global Weather Dashboard")

# --- Input ---
city = st.text_input("Enter City Name", "London")

# --- Weather Emoji Based on Conditions ---
def weather_emoji(condition):
    condition = condition.lower()
    if "cloud" in condition:
        return "☁️"
    elif "rain" in condition:
        return "🌧️"
    elif "clear" in condition:
        return "☀️"
    elif "snow" in condition:
        return "❄️"
    elif "thunder" in condition:
        return "⛈️"
    return "🌈"

# --- Get Weather Data ---
def get_weather_data(city):
    params = {'q': city, 'appid': API_KEY, 'units': 'metric'}
    res = requests.get(BASE_URL, params=params)
    return res.json() if res.status_code == 200 else None

def get_forecast_data(city):
    params = {'q': city, 'appid': API_KEY, 'units': 'metric'}
    res = requests.get(FORECAST_URL, params=params)
    return res.json() if res.status_code == 200 else None

# --- Local Time with Timezone ---
def get_local_time(lat, lon, timestamp):
    tf = TimezoneFinder()
    tz_name = tf.timezone_at(lng=lon, lat=lat)
    if tz_name:
        tz = pytz.timezone(tz_name)
        local_time = datetime.datetime.fromtimestamp(timestamp, tz)
        return local_time.strftime('%I:%M %p'), tz_name
    return "N/A", "Unknown"

# --- Main Logic ---
if st.button("Get Weather"):
    data = get_weather_data(city)

    if data:
        forecast = get_forecast_data(city)
        country_code = data['sys']['country'].lower()
        lat, lon = data['coord']['lat'], data['coord']['lon']

        st.subheader(f"📍 Weather in {data['name']}, {data['sys']['country']}")

        # --- Flag ---
        st.image(FLAG_URL.format(code=country_code), width=80)

        # --- Weather Info ---
        description = data['weather'][0]['description'].title()
        emoji = weather_emoji(description)
        st.metric("🌡️ Temperature", f"{data['main']['temp']}°C", help=description)
        st.write(f"{emoji} **Condition:** {description}")
        st.write(f"💧 **Humidity:** {data['main']['humidity']}%")
        st.write(f"💨 **Wind Speed:** {data['wind']['speed']} m/s")

        # --- Weather Icon ---
        icon_url = f"http://openweathermap.org/img/wn/{data['weather'][0]['icon']}@2x.png"
        st.image(icon_url, width=100)

        # --- Local Time Info ---
        sunrise, tz_name = get_local_time(lat, lon, data['sys']['sunrise'])
        sunset, _ = get_local_time(lat, lon, data['sys']['sunset'])
        current, _ = get_local_time(lat, lon, data['dt'])
        st.write(f"⏰ **Local Time:** {current} ({tz_name})")
        st.write(f"🌅 **Sunrise:** {sunrise}")
        st.write(f"🌇 **Sunset:** {sunset}")

        # --- Map ---
        st.map({'lat': [lat], 'lon': [lon]})

        # --- Forecast Chart ---
        if forecast:
            st.subheader("📊 5-Day Forecast (Every 3 Hours)")
            times, temps = [], []

            for entry in forecast['list'][:40]:  # next ~5 days
                timestamp = entry['dt']
                temp = entry['main']['temp']
                time_str, _ = get_local_time(lat, lon, timestamp)
                times.append(time_str)
                temps.append(temp)

            fig = go.Figure()
            fig.add_trace(go.Scatter(x=times, y=temps, mode='lines+markers', name='Temp (°C)', line=dict(color='royalblue')))
            fig.update_layout(title="Temperature Forecast", xaxis_title="Time", yaxis_title="°C", template="plotly_white")
            st.plotly_chart(fig, use_container_width=True)
    else:
        st.error("City not found or API limit exceeded.")

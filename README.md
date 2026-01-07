# ha.romierihome
Home Assistant

## 🌦️ Weather Integration

The system uses **Met.no** (Norwegian Meteorological Institute) for weather forecasting:
- ✅ Built-in Home Assistant integration (no HACS required)
- ✅ Free, reliable, global coverage
- ✅ 7-day forecast with precipitation predictions
- ✅ Updates automatically

### Setup Instructions

1. **Add your coordinates** to `config/secrets.yaml`:
   ```yaml
   latitude: YOUR_LATITUDE
   longitude: YOUR_LONGITUDE
   ```
   Find your coordinates on [LatLong.net](https://www.latlong.net/)

2. **Restart Home Assistant**

3. **Enable Smart Irrigation** (optional):
   - Go to Settings → Entities
   - Find `input_boolean.smart_irrigation_enabled`
   - Turn it ON

### Smart Irrigation Features

The weather integration enables automatic irrigation optimization:

- 🚫 **Automatic Skip**: Irrigation is blocked if >5mm rain is forecasted today
- 📱 **Morning Briefing**: Daily notification at 06:00 with weather and irrigation plan
- 📊 **Smart Factor**: Adjusts irrigation times based on:
  - Current weather conditions (rainy, cloudy, sunny)
  - Temperature (increases watering when hot)
  - Humidity (reduces watering when humid)
  - Rain forecasts (blocks watering before rain)

### Available Weather Sensors

| Sensor | Description | Unit |
|--------|-------------|------|
| `weather.romieriweather` | Main weather entity | - |
| `sensor.rain_forecast_today` | Rain forecasted today | mm |
| `sensor.rain_forecast_tomorrow` | Rain forecasted tomorrow | mm |
| `sensor.rain_forecast_next_3_days` | Total rain next 3 days | mm |
| `sensor.smart_irrigation_weather_factor` | Calculated irrigation factor | % |
| `sensor.irrigation_combined_factor` | Final irrigation factor (manual × weather) | % |
| `sensor.smart_irrigation_status` | Human-readable irrigation status | - |

### How Automatic Skip Works

Every morning at **05:00**, the system checks rain forecast:
- If >5mm rain is predicted today → Irrigation factor set to 0% (blocked)
- Notification sent explaining why irrigation was skipped
- System automatically re-enables when forecast improves

You can still manually override by adjusting `input_number.irrigation_global_factor`.


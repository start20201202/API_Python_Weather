# API_Python_Weather

https://datastudio.google.com/reporting/5585a210-7a66-464e-b90d-41002d452930/page/LMpzF/edit

https://docs.google.com/spreadsheets/d/1KjuLGQWB8Yvy8gX9SyXRfpAH7P23xF_ncEmMKJ61GvU/edit?gid=0#gid=0

Відкрийте Планувальник завдань

Натисніть Win + R → введіть taskschd.msc → Enter

```
import requests
import gspread
from oauth2client.service_account import ServiceAccountCredentials
from datetime import datetime, timedelta

CREDENTIALS_PATH = r'D:\Lehka\credentials.json'

def connect_google_sheets(sheet_name):
    scope = ["https://spreadsheets.google.com/feeds", "https://www.googleapis.com/auth/drive"]
    creds = ServiceAccountCredentials.from_json_keyfile_name(CREDENTIALS_PATH, scope)
    client = gspread.authorize(creds)
    return client.open(sheet_name).sheet1

def get_weather_history():
    cities = {
        "Kyiv": {"lat": 50.45, "lon": 30.52},
        "Lviv": {"lat": 49.84, "lon": 24.03},
        "Odesa": {"lat": 46.48, "lon": 30.72},
        "Kharkiv": {"lat": 50.00, "lon": 36.23},
        "Dnipro": {"lat": 48.46, "lon": 35.04}
    }

    end_date = datetime.now().date() - timedelta(days=1)
    start_date = end_date - timedelta(days=30)

    try:
        sheet = connect_google_sheets("Weather_Data")
        sheet.clear()
        sheet.append_row(["city", "date", "max", "min", "precipitation"])
    except Exception as e:
        print(f"Критична помилка підключення до Google Sheets: {e}")
        return

    all_rows = []

    for city_name, coords in cities.items():
        print(f"Отримання даних для міста: {city_name}...")

        url = "https://archive-api.open-meteo.com/v1/archive"
        params = {
            "latitude": coords["lat"],
            "longitude": coords["lon"],
            "start_date": start_date.strftime('%Y-%m-%d'),
            "end_date": end_date.strftime('%Y-%m-%d'),
            "daily": "temperature_2m_max,temperature_2m_min,precipitation_sum",
            "timezone": "Europe/Berlin"
        }

        response = requests.get(url, params=params)

        if response.status_code == 200:
            data = response.json()['daily']
            for i in range(len(data['time'])):
                date_obj = datetime.strptime(data['time'][i], '%Y-%m-%d')
                date_str = date_obj.strftime('%Y%m%d')  # формат YYYYMMDD
                row = [
                    city_name,
                    date_str,
                    data['temperature_2m_max'][i],
                    data['temperature_2m_min'][i],
                    data['precipitation_sum'][i]
                ]
                all_rows.append(row)
            print(f"Дані для {city_name} успішно зібрано.")
        else:
            print(f"Помилка API для {city_name}: {response.status_code}")

    if all_rows:
        print(f"\nЗапис {len(all_rows)} рядків у Google Sheets...")
        sheet.append_rows(all_rows)
        print("Готово! Перевірте вашу таблицю.")
    else:
        print("Дані для запису відсутні.")

if __name__ == "__main__":
    get_weather_history()
```

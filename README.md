import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline  # Use in Jupyter

# Load data (replace with local path after download)
traffic = pd.read_csv('Metro_Interstate_Traffic_Volume.csv')

# Convert to datetime
traffic['date_time'] = pd.to_datetime(traffic['date_time'])

# Split day (7AM-7PM) vs night
day = traffic[(traffic['date_time'].dt.hour >= 7) & (traffic['date_time'].dt.hour < 19)].copy()
night = traffic[(traffic['date_time'].dt.hour >= 19) | (traffic['date_time'].dt.hour < 7)].copy()

# Day vs Night histograms
plt.figure(figsize=(11,3.5))
plt.subplot(1, 2, 1)
plt.hist(day['traffic_volume'])
plt.xlim(-100, 7500)
plt.ylim(0, 8000)
plt.title('Traffic Volume: Day')
plt.ylabel('Frequency')
plt.xlabel('Traffic Volume')

plt.subplot(1, 2, 2)
plt.hist(night['traffic_volume'])
plt.xlim(-100, 7500)
plt.ylim(0, 8000)
plt.title('Traffic Volume: Night')
plt.ylabel('Frequency')
plt.xlabel('Traffic Volume')
plt.show()

# Day of week patterns (daytime only)
day['dayofweek'] = day['date_time'].dt.dayofweek
by_dayofweek = day.groupby('dayofweek').mean(numeric_only=True)
days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
plt.plot(by_dayofweek['traffic_volume'])
plt.xticks(range(len(days)), days)
plt.xlabel('Day of Week')
plt.ylabel('Traffic Volume')
plt.title('Traffic by Day of Week')
plt.show()

# Hourly patterns: weekdays vs weekends
day['hour'] = day['date_time'].dt.hour
business_days = day[day['dayofweek'] <= 4].copy()
weekend = day[day['dayofweek'] >= 5].copy()
by_hour_business = business_days.groupby('hour').mean(numeric_only=True)
by_hour_weekend = weekend.groupby('hour').mean(numeric_only=True)

plt.figure(figsize=(11,3.5))
plt.subplot(1, 2, 1)
plt.plot(by_hour_business['traffic_volume'])
plt.xlim(6,20)
plt.ylim(1500,6500)
plt.title('Traffic Volume By Hour: Monday–Friday')

plt.subplot(1, 2, 2)
plt.plot(by_hour_weekend['traffic_volume'])
plt.xlim(6,20)
plt.ylim(1500,6500)
plt.title('Traffic Volume By Hour: Weekend')
plt.show()

# Weather correlations (daytime)
weather_cols = ['clouds_all', 'snow_1h', 'rain_1h', 'temp', 'traffic_volume']
correlations = day[weather_cols].corr()['traffic_volume'].sort_values()
print(correlations)

# Weather main bar plot
by_weather_main = day.groupby('weather_main').mean(numeric_only=True).sort_values('traffic_volume')
plt.figure(figsize=(10,6))
plt.barh(by_weather_main.index, by_weather_main['traffic_volume'])
plt.axvline(x=5000, linestyle="--", color="k")
plt.title('Traffic Volume by Weather Type')
plt.show()

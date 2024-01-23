# Task 1

Task 1 was pretty straightforward: read a SQLite database and find the relevant locations. I wrote a basic Python script to automate this, since I figured it'd be more fun than doing it manually.

A bit of hardcoding, but it works as a bodged solution.

```python
import sqlite3
conn = sqlite3.connect('database.db')

cursor = conn.cursor()

cursor.execute('SELECT * FROM location')
location_rows = cursor.fetchall()

cursor.execute('SELECT * FROM timestamp')
time_rows = cursor.fetchall()

goal_lat = 26.39925
goal_long = -83.72485

goal_time = '11:26:58'.split(':')
goal_date = '01/31/2023'

goal_ts = int(goal_time[0]) * 3600 + int(goal_time[1]) * 60 + int(goal_time[2])

hits = []
for row in location_rows:
    index = row[0] - 1

    lat = float(row[1])
    long = float(row[2])
    
    time_arr = time_rows[index][1].split(':')
    date = time_rows[index][2]
    
    ts = int(time_arr[0]) * 3600 + int(time_arr[1]) * 60 + int(time_arr[2])
    
    lat_diff = abs(goal_lat - lat)
    long_diff = abs(goal_long - long)
    
    ts_diff = abs(ts - goal_ts)
    if (lat_diff <= 0.01 or long_diff <= 0.01) and goal_date == date:
        hits.append(index + 1)
print(hits)```
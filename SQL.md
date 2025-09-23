<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/3ffc46c1-e690-431b-84f0-110a0f0b4cd6" />




# Define user profiles for personalization in BigQuery

## We will create some user metrics for our analysis.
➡️ **avg_duration_length** → mean minutes per test group 
➡️ **skip_rate** → song skip rate %
➡️ **genre_popularity** → most frequently played genre  
➡️ **songs_per_day** → daily listening frequency  
➡️ **recency** → days since last session  

*Note: Truncated BigQuery file format*
***
### Average Duration Length (mins.) by test group 
````sql
SELECT 
  e.group AS exp_group,
  COUNT(e.user_id) AS num_users,
  ROUND(AVG(l.duration_listened)/60,2) AS avg_duration_listened_mins
FROM spotify.listening_events l
LEFT JOIN spotify.experiments e
  ON l.user_id = e.user_id 
GROUP BY exp_group;
````

<img width="561" height="83" alt="Screenshot 2025-09-22 at 9 27 14 PM" src="https://github.com/user-attachments/assets/103f8689-0d26-47f9-bb7e-c75704555a8b" />

Both groups have an average listening duration of 1.81 minutes.

***
### Skip Rate %
````sql
SELECT 
  song_id,
  ROUND(AVG(CASE WHEN skipped = TRUE THEN 1 ELSE 0 END), 2) AS skip_rate
FROM spotify.listening_events
GROUP BY song_id
ORDER BY skip_rate DESC
LIMIT 10;
````

<img width="314" height="299" alt="Screenshot 2025-09-22 at 9 37 39 PM" src="https://github.com/user-attachments/assets/e28eaec1-d1c7-4563-838e-0bf2733fbe96" />

Song 1370 has the highest skip rate of 61%. It can't be THAT bad... right?
***
### Genre Popularity
````sql
SELECT
  genre,
  num_plays,
  RANK() OVER(ORDER BY num_plays DESC) AS genre_popularity_rank
FROM (
  SELECT
    s.genre,
    COUNT(l.song_id) AS num_plays
  FROM
    spotify.listening_events l
  LEFT JOIN spotify.songs s
    ON l.song_id = s.song_id
  GROUP BY s.genre
)
ORDER BY genre_popularity_rank;
````

Classical genre is the most popular among the users.

<img width="542" height="300" alt="Screenshot 2025-09-22 at 10 13 33 PM" src="https://github.com/user-attachments/assets/9c5269b1-78c6-4b6b-b445-4168b7d4fb05" />

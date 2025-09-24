<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/3ffc46c1-e690-431b-84f0-110a0f0b4cd6" />




# Define user profiles for personalization in BigQuery

## We will create some user metrics for our analysis.
➡️ **avg_session_length** → mean minutes per user  
➡️ **skip_rate** → song skip rate % per user  
➡️ **genre_popularity** → most frequently played genre  
➡️ **songs_per_day** → daily listening frequency per user   
➡️ **recency** → days since last session  

*Note: Truncated BigQuery file format*
***
### Average Session Length per user
````sql
SELECT 
  user_id,
  ROUND(AVG(duration_listened)/60, 2) AS avg_session_length
FROM spotify.listening_events
GROUP BY user_id
ORDER BY avg_session_length DESC;
````
<img width="330" height="433" alt="Screenshot 2025-09-23 at 10 36 37 PM" src="https://github.com/user-attachments/assets/8f036161-1451-404a-87ae-2a88858745e7" />

User 2837 has the highest average session length with 4.56 mins.

***
### Skip Rate % per user
````sql
SELECT 
  user_id,
  COUNT(song_id) AS num_songs,
  ROUND(AVG(CASE WHEN skipped = TRUE THEN 1 ELSE 0 END), 2) AS skip_rate
FROM spotify.listening_events
GROUP BY user_id
ORDER BY skip_rate DESC;
````
<img width="435" height="433" alt="Screenshot 2025-09-23 at 10 39 21 PM" src="https://github.com/user-attachments/assets/b9c3a6d1-9585-4dce-8618-ba64690ecd39" />

Top four users with 100% skip rate have only listened to 4-5 songs. It's possible that the users came into Spotify with a free trial and did not enjoy it.

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

<img width="542" height="300" alt="Screenshot 2025-09-22 at 10 13 33 PM" src="https://github.com/user-attachments/assets/9c5269b1-78c6-4b6b-b445-4168b7d4fb05" />

Classical genre is the most popular among the users.
***

### Average Daily Listening Frequency per user

````sql
SELECT
  user_id,
  DATE_DIFF(MAX(timestamp), MIN(timestamp), DAY) AS num_days,
  ROUND(SUM(num_songs)/DATE_DIFF(MAX(timestamp), MIN(timestamp), DAY), 2) AS avg_daily_listening_frequency
FROM (
  SELECT
    user_id,
    timestamp,
    COUNT(song_id) AS num_songs
  FROM spotify.listening_events
  GROUP BY user_id, timestamp
)
GROUP BY user_id
HAVING DATE_DIFF(MAX(timestamp), MIN(timestamp), DAY) > 0
ORDER BY avg_daily_listening_frequency DESC;
````
<img width="437" height="434" alt="Screenshot 2025-09-23 at 10 09 35 PM" src="https://github.com/user-attachments/assets/2e454916-724e-4094-9ab8-0b6c989d1275" />

➖ Average was calculated as the number of songs listened to by the user and the number of days between the last timestamp listened and the first timestamp listened. User 2614 has the highest average daily listening frequency. However, the user has only listened over the course of four days.  
➖ This requires further analysis if highest average daily listening frequency was due to chance which will be covered in Python, i.e. Chi-Squared Test.  
➖ The results were also sorted to omit where number of days was 0. This could be a new user listening to Spotify for the first time via their free trial and not listening again.

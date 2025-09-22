<img width="200" height="200" alt="image" src="https://github.com/user-attachments/assets/3ffc46c1-e690-431b-84f0-110a0f0b4cd6" />




# Define user profiles for personalization in BigQuery

## We will create some user metrics for our analysis.
➡️ **avg_session_length** → mean minutes per session  
➡️ **skip_rate** → % skipped songs  
➡️ **genre_pref** → most frequently played genre  
➡️ **songs_per_day** → daily listening frequency  
➡️ **recency** → days since last session  

*Note: Truncated BigQuery file format*
***
### Average Duration Length by test group (mins) 
````sql
SELECT 
  e.group AS exp_group,
  COUNT(e.user_id) AS num_users,
  ROUND(AVG(l.duration_listened)/60,2) AS avg_duration_listened
FROM listening_events l
LEFT JOIN experiments e
  ON l.user_id = e.user_id 
GROUP BY exp_group
````

<img width="547" height="85" alt="Screenshot 2025-09-21 at 11 21 19 PM" src="https://github.com/user-attachments/assets/fcba5fa0-0997-4559-8caa-672a1674a783" />

Both groups have an average duration length of 1.81 minutes.

***

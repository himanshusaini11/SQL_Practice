## Question 1: Demographics and travel patterns
### (a) Which cross-section of age and gender travels the most.
~~~~sql
SELECT
	usr.user_id,
  usr.gender,
	EXTRACT(YEAR FROM AGE(CURRENT_DATE, usr.birthdate)) AS age,
  COUNT(s.flight_booked) AS num_times_travel
FROM users AS usr
LEFT JOIN sessions AS s
	ON s.user_id = usr.user_id
WHERE s.flight_booked = true
	AND s.cancellation = false
GROUP BY usr.user_id, usr.gender
ORDER BY num_times_travel DESC
LIMIT 10;
~~~~

Output: 
| user_id | gender | age | num_times_travel |
| ------- | ------ | --- | ---------------- |
| 54318   | F      | 50  | 11               |
| 50009   | F      | 39  | 11               |
| 29241   | F      | 50  | 11               |
| 144345  | F      | 38  | 11               |
| 26048   | F      | 41  | 11               |
| 12684   | F      | 39  | 10               |
| 44403   | F      | 53  | 10               |
| 1469    | F      | 48  | 10               |
| 807     | F      | 41  | 10               |
| 7030    | M      | 47  | 10               |

### (b) How does the travel behavior of customers married with children compare to childless single customers?
~~~~sql
-- First I calculated the num of trips each group made per destination.
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  GROUP BY f.destination
),
t2 AS (
  SELECT
    t1.destination,
    COALESCE(SUM(t1.num_mwc), 0) AS num_mwc,
    COALESCE(SUM(t1.num_swtc), 0) AS num_swtc,
    COALESCE(SUM(t1.num_oth), 0) AS num_oth
  FROM t1
  GROUP BY t1.destination
  ORDER BY t1.destination
)
SELECT
  t2.destination,
  t2.num_mwc,
  t2.num_swtc,
  t2.num_oth
FROM t2;
~~~~
Then, I have calculated the conversion rate of each group.
abbrevations I used.
mwc ---> Married With Childern
swtc ---> Singles Without Childeren
oth ---> Otherwise
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  GROUP BY f.destination
),
t3 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS tot_num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS tot_num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS tot_num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  GROUP BY f.destination
)
SELECT
  CAST(SUM(t1.num_mwc)/SUM(t3.tot_num_mwc)*100 AS DECIMAL(100, 3)) AS conv_mwc,
  CAST(SUM(t1.num_swtc)/SUM(t3.tot_num_swtc)*100 AS DECIMAL(100, 3)) AS conv_swtc,
  CAST(SUM(t1.num_oth)/SUM(t3.tot_num_oth)*100 AS DECIMAL(100, 3)) AS conv_oth
FROM t1, t3;
~~~~

OUTPUT:

| conv_mwc | conv_swc | conv_oth |
| -------- | -------- | -------- |
| 34.254   | 36.597   | 34.497   |


Conversion rate with other combinations.
Conversion Rate with Flight Discount = True and Hotel Discount = True
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  	AND s.flight_discount = true AND s.hotel_discount = true
  GROUP BY f.destination
),
t3 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS tot_num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS tot_num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS tot_num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  GROUP BY f.destination
)
SELECT
  CAST(SUM(t1.num_mwc)/SUM(t3.tot_num_mwc)*100 AS DECIMAL(100, 3)) AS conv_mwc,
  CAST(SUM(t1.num_swtc)/SUM(t3.tot_num_swtc)*100 AS DECIMAL(100, 3)) AS conv_swtc,
  CAST(SUM(t1.num_oth)/SUM(t3.tot_num_oth)*100 AS DECIMAL(100, 3)) AS conv_oth
FROM t1, t3;
~~~~

OUTPUT:

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 0.791    | 0.827     | 0.787    |

Conversion Rate with Flight Discount = True and Hotel Discount = False
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  	AND s.flight_discount = true AND s.hotel_discount = false
  GROUP BY f.destination
),
t3 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS tot_num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS tot_num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS tot_num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  GROUP BY f.destination
)
SELECT
  CAST(SUM(t1.num_mwc)/SUM(t3.tot_num_mwc)*100 AS DECIMAL(100, 3)) AS conv_mwc,
  CAST(SUM(t1.num_swtc)/SUM(t3.tot_num_swtc)*100 AS DECIMAL(100, 3)) AS conv_swtc,
  CAST(SUM(t1.num_oth)/SUM(t3.tot_num_oth)*100 AS DECIMAL(100, 3)) AS conv_oth
FROM t1, t3;
~~~~

OUTPUT:

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 4.259    | 4.575     | 4.274    |

Conversion Rate with Flight Discount = False and Hotel Discount = True
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  	AND s.flight_discount = False AND s.hotel_discount = true
  GROUP BY f.destination
),
t3 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS tot_num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS tot_num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS tot_num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  GROUP BY f.destination
)
SELECT
  CAST(SUM(t1.num_mwc)/SUM(t3.tot_num_mwc)*100 AS DECIMAL(100, 3)) AS conv_mwc,
  CAST(SUM(t1.num_swtc)/SUM(t3.tot_num_swtc)*100 AS DECIMAL(100, 3)) AS conv_swtc,
  CAST(SUM(t1.num_oth)/SUM(t3.tot_num_oth)*100 AS DECIMAL(100, 3)) AS conv_oth
FROM t1, t3;
~~~~

OUTPUT:
| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 3.656    | 3.887     | 3.690    |

Conversion Rate with Flight Discount = False and Hotel Discount = False
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  	AND s.flight_discount = false AND s.hotel_discount = false
  GROUP BY f.destination
),
t3 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS tot_num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS tot_num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS tot_num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  GROUP BY f.destination
)
SELECT
  CAST(SUM(t1.num_mwc)/SUM(t3.tot_num_mwc)*100 AS DECIMAL(100, 3)) AS conv_mwc,
  CAST(SUM(t1.num_swtc)/SUM(t3.tot_num_swtc)*100 AS DECIMAL(100, 3)) AS conv_swtc,
  CAST(SUM(t1.num_oth)/SUM(t3.tot_num_oth)*100 AS DECIMAL(100, 3)) AS conv_oth
FROM t1, t3;
~~~~

OUTPUT:
| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 25.549   | 27.308    | 25.747   |

No of Trip between the user groups
~~~~sql
WITH t1 AS (
  SELECT
    f.destination,
    COUNT(CASE WHEN usr.married = true AND usr.has_children = true THEN usr.user_id END) AS num_mwc,
    COUNT(CASE WHEN usr.married = false AND usr.has_children = false THEN usr.user_id END) AS num_swtc,
    COUNT(CASE WHEN NOT (usr.married = true AND usr.has_children = true OR usr.married = false AND usr.has_children = false) THEN usr.user_id END) AS num_oth
  FROM users AS usr
  LEFT JOIN sessions AS s ON s.user_id = usr.user_id
  LEFT JOIN flights AS f ON f.trip_id = s.trip_id
  WHERE s.flight_booked = true AND s.cancellation = false
  GROUP BY f.destination
),
t2 AS (
  SELECT
    t1.destination,
    COALESCE(SUM(t1.num_mwc), 0) AS num_mwc,
    COALESCE(SUM(t1.num_swtc), 0) AS num_swtc,
    COALESCE(SUM(t1.num_oth), 0) AS num_oth
  FROM t1
  GROUP BY t1.destination
  ORDER BY t1.destination
)
SELECT
  SUM(t2.num_mwc) AS num_trips_mwc,
  SUM(t2.num_swtc) AS num_trips_swtc,
  SUM(t2.num_oth) AS num_trips_oth
FROM t2;
~~~~

OUTPUT:

| num_trips_mwc | num_trips_swtc | num_trips_oth |
| ------------- | -------------- | ------------- |
| 284936        | 883001         | 733101        |

Revenue generated by the group
~~~~sql
SELECT
	CASE
  	WHEN usr.married = true AND usr.has_children = true THEN 'Married with children'
    WHEN usr.married = false AND usr.has_children = false THEN 'Single without children'
    ELSE 'Others'
  END AS usr_status,
  SUM(EXTRACT(DAY FROM(f.return_time - f.departure_time)))AS num_days_trip,
  SUM(CAST(f.base_fare_usd AS DECIMAL(100, 3))) AS revenue
FROM users AS usr
LEFT JOIN sessions AS s
	ON s.user_id = usr.user_id
LEFT JOIN flights AS f
	ON f.trip_id = s.trip_id
WHERE s.flight_booked = true
	AND s.cancellation = false
GROUP BY usr_status;
~~~~

OUTPUT:
| usr_status              | num_days_trip | revenue       |
| ----------------------- | ------------- | ------------- |
| Married with children   | 1437185       | 274743381.040 |
| Others                  | 3674940       | 456094158.020 |
| Single without children | 4315705       | 494150840.890 |


Summary of the Results
1. Conversion rate.

i. Overall Conversion

| conv_mwc | conv_swc | conv_oth |
| -------- | -------- | -------- |
| 34.254   | 36.597   | 34.497   |
   
ii. Conversion Rate with ```Flight Discount = True``` and ```Hotel Discount = True```

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 0.791    | 0.827     | 0.787    |

iii. Conversion Rate with ```Flight Discount = True``` and ```Hotel Discount = False```

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 4.259    | 4.575     | 4.274    |

iv. Conversion Rate with ```Flight Discount = False``` and ```Hotel Discount = True```

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 3.656    | 3.887     | 3.690    |

v. Conversion Rate with ```Flight Discount = False``` and ```Hotel Discount = False```

| conv_mwc | conv_swtc | conv_oth |
| -------- | --------- | -------- |
| 25.549   | 27.308    | 25.747   |

3. Number of Trips the group made.

| num_trips_mwc | num_trips_swtc | num_trips_oth |
| ------------- | -------------- | ------------- |
| 284936        | 883001         | 733101        |

4. Number of days they spend during the Trips the group made with the total revenue they have generated.

| usr_status              | num_days_trip | revenue       |
| ----------------------- | ------------- | ------------- |
| Married with children   | 1437185       | 274743381.040 |
| Others                  | 3674940       | 456094158.020 |
| Single without children | 4315705       | 494150840.890 |

Discussion: Based on a comprehensive analysis of conversion rates and user behavior, it is evident that providing discounts does not significantly impact the conversion of users. A substantial portion of conversions is observed within the category of users who did not avail any discounts. Notably, the majority of trips are taken by individuals without children, emphasizing the importance of tailoring trip packages specifically for this demographic.

Furthermore, an additional analysis based on countries was initiated, though not explored in detail due to the potential length of the analysis. However, a preliminary examination indicated that trips were most frequently taken across various continents. To refine this insight, subsequent analyses delved into the number of days spent and the resulting revenue. Remarkably, individuals classified as singles without children emerged as the primary contributors to overall revenue.

In light of these findings, it is recommended that the team strategically focuses on crafting and promoting enticing trip packages aimed at attracting more users from the singles without children category. The data-driven approach suggests that this demographic not only engages in frequent travel but also contributes significantly to the revenue stream. By aligning marketing efforts and trip offerings with the preferences and behaviors of this user segment, the team can capitalize on the lucrative opportunities presented by their travel patterns. This targeted approach is poised to enhance user engagement, increase conversions, and ultimately drive greater revenue for the travel platform.


## Question 2: Session analysis
### (a) How much session abandonment do we see? Session abandonment means they browsed but did not book anything.
~~~~sql
SELECT
	COUNT(usr.user_id) AS session_aband
FROM users AS usr
LEFT JOIN sessions AS s
	ON s.user_id = usr.user_id
WHERE s.flight_booked = FALSE AND s.hotel_booked = FALSE;
~~~~

OUTPUT:

| session_aband |
| ------------- |
| 3072218       |

### (b) Which demographics abandon sessions disproportionately more than average?
~~~~sql
SELECT
	CASE
  	WHEN usr.married = true AND usr.has_children = true THEN 'Married with children'
    WHEN usr.married = false AND usr.has_children = false THEN 'Single without children'
    ELSE 'Others'
  END AS usr_status,
	COUNT(usr.user_id) AS session_aband
FROM users AS usr
LEFT JOIN sessions AS s
	ON s.user_id = usr.user_id
WHERE s.flight_booked = FALSE AND s.hotel_booked = FALSE
GROUP BY usr_status;
~~~~

Output:

| usr_status              | session_aband |
| ----------------------- | ------------- |
| Married with children   | 484260        |
| Others                  | 1231946       |
| Single without children | 1356012       |


## Question3: Geographic influences on travel preference
### (a) Explore how customer origin (e.g. home city) influences travel preferences.
~~~~sql
WITH DataSet AS (
  SELECT
    usr.user_id,
    usr.home_city,
    usr.home_airport,
    usr.home_airport_lat,
    usr.home_airport_lon,
    f.origin_airport,
    f.destination,
    f.destination_airport,
    f.destination_airport_lat,
    f.destination_airport_lon
  FROM users AS usr
  LEFT JOIN sessions AS s
    ON s.user_id = usr.user_id
  LEFT JOIN flights AS f
    ON f.trip_id = s.trip_id
  WHERE s.flight_booked = TRUE
)
SELECT
	CASE
  	WHEN ds.home_airport = ds.origin_airport THEN 'Same'
    ELSE 'Different'
  END AS is_same_city_airport,
  COUNT(ds.user_id) AS num_pass
FROM DataSet AS ds
GROUP BY is_same_city_airport;
~~~~

Output:

| is_same_city_airport | num_pass |
| -------------------- | -------- |
| Different            | 1326     |
| Same                 | 1990382  |

Note: There are much more information in ```DataSet``` table but that is for further analysis.

## Question 4: Can you make any strategic recommendations based on your answers to the questions above?
Based on what we found, it's a good idea for the team to focus on creating special travel packages for people who are single and don't have children. These folks not only travel a lot but also bring in a significant amount of money for the platform. If we tailor our marketing and trips to what they like, we can take advantage of the opportunities presented by their travel habits. This targeted approach can make users more engaged, increase the number of people who book trips, and, in the end, make more money for the travel platform.

Also, when we look at sessions that were started but not finished, we see that single users without children have the highest number. This suggests that many of these passengers didn't end up booking a trip. If the team comes up with some clever ideas to attract them, we can boost our revenue. For instance, if we assume a lot of these users are students, offering them interesting deals like cheaper flights during holidays might increase the number of bookings.

Additionally, it's crucial for the team to pay attention to customers in major airport cities because they are the ones making the most bookings. Offering attractive packages, such as cheaper cab services within a 25 KM range from the airport, could be a smart move. This could create a psychological impact, encouraging users to book both the flight and other services at the same time. While connecting the cab service might be a bit challenging, it opens up a new opportunity for the company to attract more users.

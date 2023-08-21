# SQL Practice Project on Aireline Data.
It includes the SQL queries during my learning process.
## Business case - Identifying business areas with the most flight delays

Database link: ```postgres://Test:bQNxVzJL4g6u@ep-noisy-flower-846766.us-east-2.aws.neon.tech/DA104.1``` \
It containd 4 tables:
1. ba_aircraft
2. ba_flight_routes
3. ba_filghts
4. ba_fuel_efficiency

### Question 1: What is the total number of flights that have been delayed versus not delayed for all completed flights?
-Total number of flights, that means one value for each. Hence, I have to use ```COUNT()```.
~~~~sql
SELECT
        CASE WHEN status LIKE 'Completed' AND delayed_flag LIKE 'Y' THEN 'Delayed'
                WHEN status LIKE 'Completed' AND delayed_flag LIKE 'N' THEN 'Not Delayed'
        ELSE 'Otherwise' END AS flight_status,
        COUNT(flight_id) AS num_delayed
        --status,
        --delayed_flag
FROM ba_flights
--WHERE status LIKE 'Completed' AND delayed_flag LIKE 'Y';
GROUP BY flight_status
ORDER BY num_delayed;
~~~~
Output :
| flight_status | num_delayed |
| ------------- | ----------- |
| Delayed       | 56          |
| Not Delayed   | 241         |
| Otherwise     | 715         |


### <div style="text-align: justify"> Question 2: The VP of operations suggested that the delays could be caused by the type of aircraft the passengers are boarding into. Some aircraft designs are easier to board than others. Could you look into this? How many flights, out of all flights, have been delayed per manufacturer and ac subtype? Make sure that all delayed flights, even if there is no aircraft assigned to the flight and include the capacity of for each aircraft where available. </div>
-We have to count the total number of delaed flights as per aircraft subtype and manufacturer, use ```COUNT()```.
~~~~sql
SELECT
        --baf.status,
        baa.ac_subtype,
        baa.manufacturer,
        bae.capacity,
        COUNT(baf.flight_id) AS num_delayed_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
LEFT JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype

WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'

GROUP BY baa.ac_subtype, baa.manufacturer, bae.capacity
ORDER BY num_delayed_flights DESC;

~~~~

Output :
| ac_subtype | manufacturer | capacity | num_delayed_flights |
| ---------- | ------------ | -------- | ------------------- |
|            |              |          | 14                  |
| 73H        | Boeing       | 440      | 11                  |
| 772        | Boeing       | 220      | 7                   |
| 73W        | Boeing       | 189      | 6                   |
| E90        | Embraer      | 380      | 5                   |
| 789        | Boeing       | 550      | 3                   |
| E75        | Embraer      | 88       | 3                   |
| 332        | Airbus       | 406      | 3                   |
| 73J        | Boeing       | 330      | 2                   |
| 295        | Boeing       | 400      | 2                   |


### Question 3: The head of international flights tends to be a little selfish and therefore its no surprise he wants to assess whether the international flights are not causing the delays. Could you analyse the number of delays (compared against non-delayed flights) only for the completed flights that has a capacity over 300 passengers? These airplanes tend to only be used for international flights.
~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flight_status,
	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
INNER JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype
WHERE bae.capacity > 300 AND baf.status LIKE 'Completed'
GROUP BY flight_status;
~~~~

Output :
| flight_status | num_flights |
| ------------- | ----------- |
| Not Delayed   | 131         |
| Delayed       | 26          |


### Question 4: Even though the query easily compares delayed versus not-delayed flights in question 3, we still don’t have an actual comparison against flights with a lower passenger capacity. Reuse the query of question 3, but instead of filtering for capacity over 300 passengers, create a column that segments aircraft below or equal to 300 and over 300. How do the delays in total number of flights compare between the two segments?
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flight_status,
	CASE WHEN bae.capacity <= 300 THEN 'Below or equal 300'
      	ELSE 'Above 300' END AS flight_segments,
	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
INNER JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype
WHERE baf.status LIKE 'Completed'
GROUP BY flight_status, flight_segments
ORDER BY flight_segments;
~~~~

Output :
| flight_status | flight_segments    | num_flights |
| ------------- | ------------------ | ----------- |
| Delayed       | Above 300          | 26          |
| Not Delayed   | Above 300          | 131         |
| Delayed       | Below or equal 300 | 16          |
| Not Delayed   | Below or equal 300 | 69          |


### Question 5: Your manager suggests to analyse whether specific locations are causing the delays. Are certain airports better organized than others? Let’s find out. What departure cities have experienced the highest number of delays in all the completed flights? Only include flights that operates with the manufacturers Boeing and Airbus.
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flights_status,
      	bar.departure_city,
      	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number
INNER JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'
	AND (baa.manufacturer = 'Boeing' OR baa.manufacturer = 'Airbus')
GROUP BY bar.departure_city, baf.status, baf.delayed_flag
ORDER BY num_flights DESC;
~~~~

Output :
| flights_status | departure_city | num_flights |
| -------------- | -------------- | ----------- |
| Delayed        | London         | 11          |
| Delayed        | Krakow         | 2           |
| Delayed        | Nantes         | 2           |
| Delayed        | Dammam         | 2           |
| Delayed        | Basel          | 2           |
| Delayed        | Aberdeen       | 2           |
| Delayed        | Trondheim      | 2           |
| Delayed        | Manila         | 1           |
| Delayed        | Port of Spain  | 1           |
| Delayed        | Stockholm      | 1           |
| Delayed        | São Paulo      | 1           |
| Delayed        | Linköping      | 1           |
| Delayed        | Bergen         | 1           |
| Delayed        | Dubai          | 1           |
| Delayed        | Helsinki       | 1           |
| Delayed        | Kuala Lumpur   | 1           |
| Delayed        | Aarhus         | 1           |
| Delayed        | Lisbon         | 1           |


### Question 6: It looks like we have found a potential area where delays are happening more frequently. You remember that there are some flights did not have any aircraft assigned in the data at the moment. Because of that, you would like to check whether the number of delays increases even more if we include these flights with missing aircraft assignment. What is the total number of delays of all completed flights per manufacturer that departed from London?
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		--WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
      	END AS flights_status,
      	baa.manufacturer,
      	--bar.departure_city,
      	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'
	AND bar.departure_city LIKE 'London'
GROUP BY baa.manufacturer, bar.departure_city, baf.status, baf.delayed_flag
ORDER BY num_flights DESC;
~~~~

Output :
| flights_status | manufacturer | num_flights |
| -------------- | ------------ | ----------- |
| Delayed        | Boeing       | 9           |
| Delayed        |              | 5           |
| Delayed        | Embraer      | 3           |
| Delayed        | Airbus       | 2           |


### Question 7: Create a list of flights, showing the flight ID, departure city, arrival city, manufacturer, and aircraft sub-type that will be used for each flight. Show the results for all flights that are available even if not all information is available for all flights.
~~~sql
SELECT
	baf.flight_id,
      	bar.departure_city,
      	bar.arrival_city,
      	baa.manufacturer,
      	baa.ac_subtype
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number

LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
LIMIT 10; -- To get the required answer remove the ```LIMIT```. I used just to show the output.
~~~

Output :
| flight_id | departure_city | arrival_city | manufacturer | ac_subtype |
| --------- | -------------- | ------------ | ------------ | ---------- |
| AAAA01    | Stockholm      | London       | Boeing       | 73H        |
| AAAA02    | London         | Riyadh       |              |            |
| AAAA03    | London         | Riyadh       |              |            |
| AAAA04    | Dammam         | London       |              |            |
| AAAA05    | London         | Dubai        |              |            |
| AAAA06    | Dubai          | London       |              |            |
| AAAA07    | Dubai          | London       | Airbus       | 332        |
| AAAA08    | London         | Kuwait City  | Airbus       | 332        |
| AAAA09    | London         | Kuwait City  | Airbus       | 332        |
| AAAA10    | Muscat         | Kuwait City  | Airbus       | 332        |


### Question 8: What is the maximum number of passengers that have been on every available aircraft (manufacturer and sub-type) for flights that have been completed? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
      baa.manufacturer,
      baa.ac_subtype,
      MAX(baf.total_passengers) AS max_passengers
FROM ba_flights AS baf
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
WHERE baf.status = 'Completed'
GROUP BY baa.manufacturer, baa.ac_subtype
ORDER BY max_passengers;
~~~~

Output :
| manufacturer | ac_subtype | max_passengers |
| ------------ | ---------- | -------------- |
| Boeing       | 73W        | 264            |
| Boeing       | 789        | 264            |
| Embraer      | E90        | 277            |
| Boeing       | 73H        | 326            |
| Boeing       | 772        | 373            |
| Airbus       | 332        | 373            |
| Boeing       | 295        | 373            |
| Embraer      | E75        | 376            |
| Boeing       | 73J        | 376            |


### Question 9: Since only some aircraft are capable of flying long distances overseas, we want to filter out the planes that only do shorter distances. What aircraft (manufacturer and sub-type) have completed flights of a distance of more than 7,000 km? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
      DISTINCT baa.manufacturer,
      baa.ac_subtype,
      SUM(bar.distance_flown) AS distance_flown
FROM ba_flights AS baf
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
INNER JOIN ba_flight_routes AS bar
ON bar.flight_number = baf.flight_number
WHERE baf.status = 'Completed'
	AND bar.distance_flown > 7000
GROUP BY baa.manufacturer, baa.ac_subtype;
~~~~

Output :
| manufacturer | ac_subtype | distance_flown |
| ------------ | ---------- | -------------- |
| Airbus       | 332        | 19550          |
| Boeing       | 295        | 17624          |
| Boeing       | 73H        | 18587          |
| Boeing       | 772        | 19102          |
| Boeing       | 789        | 28757          |
| Embraer      | E90        | 9775           |


### Question 10: What is the most used aircraft (manufacturer and sub-type) for flights departing from London and arriving in Basel, Trondheim, or Glasgow? Include the number of flights that the aircraft was used for. If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
      baa.ac_subtype,
      baa.manufacturer,
      COUNT(baf.flight_id) AS num_aircrafts
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON bar.flight_number = baf.flight_number
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
WHERE bar.departure_city LIKE 'London'
	AND bar.arrival_city IN ('Basel', 'Trondheim', 'Glasgow')
GROUP BY baa.ac_subtype, baa.manufacturer
ORDER BY num_aircrafts DESC
LIMIT 1;
~~~~

Output :
| ac_subtype | manufacturer | num_aircrafts |
| ---------- | ------------ | ------------- |
| 295        | Boeing       | 9             |


### Question 11: For the flight routes highlighted in question 4 combined, would there have been an aircraft that, on average, would use less fuel on the flight routes? The fuel used in liters per flight can be calculated by multiplying the fuel efficiency metric by distance, baggage weight, and number of passengers. What aircraft (manufacturer and sub-type) would you recommend to use for each of these flight routes if you use the average fuel consumption as your guiding metric? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~sql
SELECT
      baa.ac_subtype,
      baa.manufacturer,
      AVG(bae.fuel_efficiency * bar.distance_flown * baf.baggage_weight * baf.total_passengers) AS fuel_avg
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON bar.flight_number = baf.flight_number
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
INNER JOIN ba_fuel_efficiency AS bae
ON bae.ac_subtype = baa.ac_subtype
WHERE bar.departure_city LIKE 'London'
	AND bar.arrival_city IN ('Basel', 'Trondheim', 'Glasgow')
GROUP BY baa.ac_subtype, baa.manufacturer
ORDER BY fuel_avg;
~~~~

Output :
| ac_subtype | manufacturer | fuel_avg           |
| ---------- | ------------ | ------------------ |
| 772        | Boeing       | 2294069.9296594504 |
| 73J        | Boeing       | 3186193.86556758   |
| 789        | Boeing       | 3350866.3948811316 |
| 295        | Boeing       | 4209148.12909342   |
| 73W        | Boeing       | 5989304.848672802  |
| 73H        | Boeing       | 7874115.00743448   |
| E90        | Embraer      | 10156384.79437725  |
| E75        | Embraer      | 16032613.238630371 |
| 332        | Airbus       | 270770299.2574519  |


### Question 12: The fuel used in liters per flight can be calculated by multiplying the fuel efficiency metric by distance, baggage weight, and number of passengers. Calculate the total amount of fuel used per kilometer flown of completed flights per manufacturer. What manufacturer has used less fuel per km in total? If flights do not have data available about the aircraft type, you can exclude the flights from the analysis
~~~~sql
SELECT
  baa.manufacturer,
  SUM(
    bae.fuel_efficiency * bar.distance_flown * baf.baggage_weight * baf.total_passengers
  ) / SUM(bar.distance_flown) AS fuel_avg_per_km
FROM
  ba_flights AS baf
  LEFT JOIN ba_flight_routes AS bar ON bar.flight_number = baf.flight_number
  INNER JOIN ba_aircraft AS baa ON baa.flight_id = baf.flight_id
  INNER JOIN ba_fuel_efficiency AS bae ON bae.ac_subtype = baa.ac_subtype
WHERE
  baf.status = 'Completed'
GROUP BY
  baa.manufacturer
ORDER BY
  fuel_avg_per_km;
~~~~

Output :
| manufacturer | fuel_avg_per_km   |
| ------------ | ----------------- |
| Boeing       | 4472.67905248529  |
| Embraer      | 9853.451706778491 |
| Airbus       | 394688.612612313  |


### Question 13: To get started with analysis, create a summary of how many short-haul versus long-haul flights happen. A typical short-haul flight in Europe has a maximum distance of 2,000 km. How many flights are scheduled or completed for both short-haul and long-haul flights in 2023?
~~~~sql
SELECT
	CASE WHEN bar.distance_flown <= 2000 THEN 'short_haul'
  ELSE 'long_haul' END AS haul_category,
  COUNT(baf.flight_id) AS num_flight
FROM ba_flights AS baf
	INNER JOIN ba_flight_routes AS bar
		ON bar.flight_number = baf.flight_number
WHERE baf.status IN ('Completed', 'Scheduled')
	AND EXTRACT(YEAR FROM baf.actual_flight_date) = 2023
GROUP BY haul_category;
~~~~

Output :
| haul_category | num_flight |
| ------------- | ---------- |
| long_haul     | 118        |
| short_haul    | 536        |

### Question 14: We can calculate how full flights were by comparing the number of passengers on the flight against the capacity of the aircraft. Calculate the average number of empty seats for the short-haul and long-haul flights. Additionally, can you also calculate the average number of empty seats as a percentage of the maximum number of passengers? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
	CASE WHEN bar.distance_flown <= 2000 THEN 'short_haul'
  	ELSE 'long_haul' END AS haul_category,
	AVG(bae.capacity - baf.total_passengers) AS avg_empty_seats,
  AVG((bae.capacity - baf.total_passengers)/bae.capacity) AS avg_empty_seats_perc
FROM ba_flights AS baf
	INNER JOIN ba_aircraft AS baa
  	ON baa.flight_id = baf.flight_id
  INNER JOIN ba_flight_routes AS bar
  	ON bar.flight_number = baf.flight_number
  INNER JOIN ba_fuel_efficiency AS bae
  	ON bae.ac_subtype = baa.ac_subtype
GROUP BY haul_category;
~~~~

Output :
| haul_category | avg_empty_seats      | avg_empty_seats_perc   |
| ------------- | -------------------- | ---------------------- |
| short_haul    | 222.6193265007320644 | 0.09809663250366032211 |
| long_haul     | 229.2105263157894737 | 0.09210526315789473684 |


### Question 15: Calculate the total number of scheduled flights used with more than 100 empty seats in the plane. Split the flights by short-haul and long-haul flights. Exclude the flights where the manufacturer and sub-type are not available
~~~~sql
SELECT
	CASE WHEN bar.distance_flown <= 2000 THEN 'short_haul'
  	ELSE 'long_haul' END AS haul_category,
  COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
  LEFT JOIN ba_aircraft AS baa
  	ON baa.flight_id = baf.flight_id
  INNER JOIN ba_flight_routes AS bar
  	ON bar.flight_number = baf.flight_number
  INNER JOIN ba_fuel_efficiency AS bae
  	ON bae.ac_subtype = baa.ac_subtype
WHERE bae.capacity - baf.total_passengers > 100
	AND baf.status = 'Scheduled'
GROUP BY haul_category;
~~~~

Output :
| haul_category | num_flights |
| ------------- | ----------- |
| short_haul    | 186         |
| long_haul     | 45          |


### Question 16: What short-haul flight routes that have been completed have the highest average number of empty seats? Include the flight number, departure city, arrival city, number of completed flights, and average empty seats in your results. Make sure to include all flights that are available in the data even if the capacity information for some flights might be missing.
~~~~sql
SELECT
	baf.flight_number,
  bar.departure_city,
  bar.arrival_city,
  COUNT(baf.flight_id) AS num_flights_completed,
	AVG(bae.capacity - baf.total_passengers) AS avg_empty_seats
FROM ba_flights AS baf
	LEFT JOIN ba_aircraft AS baa
  	ON baa.flight_id = baf.flight_id
  LEFT JOIN ba_flight_routes AS bar
  	ON bar.flight_number = baf.flight_number
  LEFT JOIN ba_fuel_efficiency AS bae
  	ON bae.ac_subtype = baa.ac_subtype
WHERE bar.distance_flown <=2000
	AND baf.status = 'Completed'
 GROUP BY
 	baf.flight_number,
  bar.departure_city,
  bar.arrival_city
ORDER BY avg_empty_seats DESC;
~~~~

Output :
| flight_number | departure_city | arrival_city | num_flights_completed | avg_empty_seats      |
| ------------- | -------------- | ------------ | --------------------- | -------------------- |
| BA1997        | Lisbon         | London       | 1                     |                      |
| BA1752        | London         | Humberside   | 1                     | 550.0000000000000000 |
| BA2002        | London         | Lisbon       | 4                     | 460.0000000000000000 |
| BA1753        | Humberside     | London       | 2                     | 440.0000000000000000 |
| BA1730        | Aarhus         | London       | 3                     | 429.6666666666666667 |
| BA2000        | London         | Lisbon       | 3                     | 347.5000000000000000 |
| BA1377        | London         | Helsinki     | 1                     | 330.0000000000000000 |
| BA1371        | Gothenburg     | London       | 3                     | 322.3333333333333333 |
| BA1706        | Aberdeen       | London       | 2                     | 321.0000000000000000 |
| BA1538        | London         | Toulouse     | 3                     | 319.3333333333333333 |
| BA2359        | London         | Krakow       | 2                     | 316.5000000000000000 |
| BA933         | Port of Spain  | London       | 2                     | 315.0000000000000000 |
| BA1381        | Helsinki       | London       | 6                     | 311.6000000000000000 |
| BA1744        | Glasgow        | London       | 2                     | 305.0000000000000000 |
| BA631         | Kuala Lumpur   | Jakarta      | 2                     | 303.5000000000000000 |
| BA2342        | London         | Basel        | 7                     | 296.8000000000000000 |
| BA2006        | Madrid         | London       | 3                     | 290.0000000000000000 |
| BA2008        | Madrid         | London       | 2                     | 286.0000000000000000 |
| BA1394        | London         | Linköping    | 4                     | 280.7500000000000000 |
| BA1748        | London         | Glasgow      | 2                     | 280.0000000000000000 |
| BA2005        | London         | Madrid       | 3                     | 279.0000000000000000 |
| BA1390        | Linköping      | London       | 8                     | 277.2500000000000000 |
| BA1388        | Trondheim      | London       | 7                     | 272.6000000000000000 |
| BA1716        | Nantes         | London       | 3                     | 268.5000000000000000 |
| BA1534        | Toulouse       | London       | 5                     | 267.6666666666666667 |
| BA1382        | London         | Helsinki     | 7                     | 264.8000000000000000 |
| BA1374        | Helsinki       | London       | 5                     | 258.6000000000000000 |
| BA1729        | London         | Aarhus       | 9                     | 257.2500000000000000 |
| BA1723        | Nantes         | London       | 3                     | 254.0000000000000000 |
| BA1517        | Edinburgh      | London       | 4                     | 249.5000000000000000 |
| BA1397        | Bergen         | London       | 4                     | 246.2500000000000000 |
| BA1787        | Norwich        | London       | 5                     | 231.0000000000000000 |
| BA1762        | London         | Humberside   | 3                     | 230.5000000000000000 |
| BA2004        | Madrid         | London       | 3                     | 228.5000000000000000 |
| BA1707        | London         | Aberdeen     | 6                     | 224.6666666666666667 |
| BA1310        | Stockholm      | London       | 2                     | 220.0000000000000000 |
| BA1444        | Paris          | London       | 3                     | 208.0000000000000000 |
| BA1379        | London         | Helsinki     | 5                     | 207.5000000000000000 |
| BA2351        | Krakow         | London       | 2                     | 206.0000000000000000 |
| BA922         | Kralendijk     | London       | 6                     | 200.0000000000000000 |
| BA2007        | London         | Madrid       | 5                     | 198.8000000000000000 |
| BA2338        | London         | Budapest     | 3                     | 195.5000000000000000 |
| BA907         | Oranjestad     | Kralendijk   | 5                     | 192.2000000000000000 |
| BA1700        | London         | Aberdeen     | 4                     | 191.3333333333333333 |
| BA544         | Muscat         | Kuwait City  | 3                     | 181.0000000000000000 |
| BA893         | Quito          | Guayaquil    | 2                     | 178.5000000000000000 |
| BA2355        | Krakow         | London       | 5                     | 178.0000000000000000 |
| BA1702        | Aberdeen       | London       | 3                     | 173.0000000000000000 |
| BA1516        | London         | Edinburgh    | 6                     | 168.0000000000000000 |
| BA1775        | Valencia       | London       | 4                     | 167.5000000000000000 |
| BA2341        | Basel          | London       | 7                     | 164.4000000000000000 |
| BA1398        | London         | Bergen       | 6                     | 148.2500000000000000 |
| BA1526        | London         | Edinburgh    | 5                     | 144.7500000000000000 |
| BA1712        | London         | Aberdeen     | 2                     | 127.0000000000000000 |
| BA1771        | London         | Valencia     | 2                     | 125.5000000000000000 |
| BA1738        | London         | Glasgow      | 6                     | 107.8000000000000000 |
| BA2349        | London         | Basel        | 2                     | 107.0000000000000000 |
| BA2343        | Basel          | London       | 3                     | 92.6666666666666667  |
| BA1705        | London         | Aberdeen     | 2                     | 92.0000000000000000  |
| BA1709        | Aberdeen       | London       | 3                     | 89.3333333333333333  |
| BA721         | Kigali         | Kampala      | 3                     | 79.0000000000000000  |
| BA1510        | Edinburgh      | London       | 2                     | 51.5000000000000000  |
| BA1739        | Glasgow        | London       | 4                     | 48.3333333333333333  |
| BA2001        | Lisbon         | London       | 6                     | 48.2500000000000000  |
| BA1378        | Helsinki       | London       | 2                     | 33.0000000000000000  |
| BA1399        | Bergen         | London       | 2                     | 7.0000000000000000   |
| BA1774        | London         | Valencia     | 2                     | 6.0000000000000000   |

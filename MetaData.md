# SQL Practice Project on Meta Data.
It includes the SQL queries during my learning process.

Database link: ```postgres://Test:bQNxVzJL4g6u@ep-noisy-flower-846766.us-east-2.aws.neon.tech/DA104.2``` \
It containd 4 tables:
1. ```meta_clients```
2. ```meta_employees```
3. ```meta_offsites```
4. ```meta_revenue```

### Question 1: What is the average annual revenue per sector of clients that work in the sectors Insurance and Banking?
- First we have make sure the data is clean. After looking into the ```sales_team```, ```industry```, ```sector```, and ```region``` column need to be either upper case or lowercase but not the mixed.
~~~~sql
SELECT
	UPPER(sector) AS sector,
  AVG(annual_revenue) AS avg_annual_rev_per_sector
FROM meta_clients
WHERE LOWER(sector) IN ('insurance', 'banking')
GROUP BY UPPER(sector);
~~~~

Output :
| sector    | avg_annual_rev_per_sector |
| --------- | ------------------------- |
| BANKING   | 1364.0833333333333        |
| INSURANCE | 355.4166666666667         |


### Question 2: Humberto wants to analyze the marketing spend percentage by country, but the data is not 100% clean. He mentions that you can clean the country field by using the sales team information, and shared the following mapping with you: 
### UK = United Kingdom
### FR = France
### ES = Spain
### IT = Italy
### DACH = Germany
### What is the average marketing spend percentage per country?
~~~sql
SELECT
  CASE
  	WHEN mc.country IS NULL THEN (
      CASE
      	WHEN mc.sales_team LIKE ('%UK%') THEN 'United Kingdom'
      	WHEN mc.sales_team LIKE ('%FR%') THEN 'France'
    		WHEN mc.sales_team LIKE ('%ES%') THEN 'Spain'
    		WHEN mc.sales_team LIKE ('%IT%') THEN 'Italy'
    		WHEN mc.sales_team LIKE ('%DACH%') THEN 'Germany'
      END
      )
    ELSE mc.country
	END AS upd_country,
  AVG(mc.marketing_spend_perc) AS avg_marketing_spend_perc
FROM meta_clients AS mc
GROUP BY mc.sales_team, upd_country;
~~~

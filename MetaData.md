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
GROUP BY sector;
~~~~

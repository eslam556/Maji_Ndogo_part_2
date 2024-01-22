# Maji Ndogo

**Author**: Eslam Mohamed Hafez <br>
**Email**: em4132275@gmail.com <br>
**LinkedIn**: https://www.linkedin.com/in/eslam-mohamed-b828521a9

**1.** How many UV filters do we have to install in total?

````sql
SELECT
    Improvement,
    COUNT(*) AS totals
FROM
    Project_progress
WHERE
    Improvement == 'Install UV filter'
GROUP BY
    Improvement;
````

**Results:**

Improvement	   |    totals  |
:-------------:|:----------:|
1 January 2021 |    5374    |

**2.** Which province should we send drilling equipment to first?

````sql
WITH TEST AS(
SELECT
    province_name,
    SUM(people_served) AS total_ppl_serv
FROM
    combined_analysis_table CT
GROUP BY
    province_name)

SELECT
    CT.province_name,

    -- These case statements create columns for each type of source.
    -- The results are aggregated and percentages are calculated

    ROUND(SUM(CASE WHEN source_type = 'river'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS river,
    ROUND(SUM(CASE WHEN source_type = 'shared_tap'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS shared_tap,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home_broken'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home_broken,
    ROUND(SUM(CASE WHEN source_type = 'well' AND CT.results != 'Clean'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS well
FROM
    combined_analysis_table CT
JOIN
    TEST PT
ON
    CT.province_name = PT.province_name
GROUP BY
    CT.province_name
ORDER BY
    CT.province_name;
````

**Results:**

province_name   |   river   |   shared_tap  |   tap_in_home  |   tap_in_home_broken  |   well   |
:--------------:|:---------:|:-------------:|:--------------:|:---------------------:|:--------:|
Akatsi	        |     5     |       49      |       14       |          10           |    13    |
Amanzi	        |     3     |       38      |       28       |          24           |    4     |
Hawassa	        |     4     |       43      |       15       |          15           |    20    |
Kilimani	    |     28    |       47      |       13       |          12           |    16    |
Sokoto	        |     21    |       38      |       16       |          10           |    11    |

**3.** Which towns should we upgrade shared taps first?

````sql
WITH TEST AS (-- This CTE calculates the population of each town
-- Since there are two Harare towns, we have to group by province_name and town_name

SELECT
    province_name,
    town_name,
    SUM(people_served) AS total_ppl_serv
FROM
    combined_analysis_table CT
GROUP BY
    province_name,
    town_name)

SELECT
    CT.province_name,
    CT.town_name,
    
    ROUND(SUM(CASE WHEN source_type = 'river'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS river,
    ROUND(SUM(CASE WHEN source_type = 'shared_tap'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS shared_tap,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home_broken'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home_broken,
    ROUND(SUM(CASE WHEN source_type = 'well'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS well
FROM
    combined_analysis_table CT
JOIN
    TEST TT
ON
    CT.province_name = TT.province_name
    AND CT.town_name = TT.town_name
GROUP BY
    CT.province_name,
    CT.town_name;
````

**Results:**

province_name   |   town_name   |   river   |   shared_tap  |   tap_in_home  |   tap_in_home_broken  |   well   |
:--------------:|:-------------:|:---------:|:-------------:|:--------------:|:---------------------:|:--------:|
Akatsi	        |     Lusaka    |     2     |       17      |       28       |          28           |    26    |
Akatsi	        |     Rural     |     6     |       59      |       9        |          5            |    22    |
Akatsi	        |     Kintampo  |     2     |       15      |       31       |          26           |    26    |
Akatsi	        |     Harare    |     2     |       17      |       28       |          27           |    27    |
Amanzi	        |     Dahabu    |     3     |       37      |       55       |          1            |    4     |

**4.** What is the maximum percentage of the population using rivers in a single town in the Amanzi province?

````sql
WITH TEST AS (-- This CTE calculates the population of each town
-- Since there are two Harare towns, we have to group by province_name and town_name

SELECT
    province_name,
    town_name,
    SUM(people_served) AS total_ppl_serv
FROM
    combined_analysis_table CT
GROUP BY
    province_name,
    town_name)

SELECT
    CT.province_name,
    CT.town_name,
    
    ROUND(SUM(CASE WHEN source_type = 'river'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS river,
    ROUND(SUM(CASE WHEN source_type = 'shared_tap'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS shared_tap,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home,
    ROUND(SUM(CASE WHEN source_type = 'tap_in_home_broken'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS tap_in_home_broken,
    ROUND(SUM(CASE WHEN source_type = 'well'
        THEN people_served ELSE 0 END) * 100 / total_ppl_serv, 0) AS well
FROM
    combined_analysis_table CT
JOIN
    TEST TT
ON
    CT.province_name = TT.province_name
    AND CT.town_name = TT.town_name
WHERE
    CT.province_name = 'Amanzi'
GROUP BY
    CT.province_name,
    CT.town_name;
````

**Results:**

province_name   |   town_name   |	river   |	shared_tap  |	tap_in_home    | 	tap_in_home_broken  |   well    |
:--------------:|:-------------:|:---------:|:-------------:|:----------------:|:----------------------:|:---------:|
Amanzi          |     Dahabu	|     3     |   	37	    |       55	       |            1	        |    4      |
Amanzi          |     Rural 	|     3     |   	27	    |       30	       |            30	        |    10     |
Amanzi          |     Abidjan	|     2     |   	53	    |       22	       |            19	        |    4      |
Amanzi          |     Pwani	    |     3     |   	53	    |       20	       |            21	        |    4      |
Amanzi          |     Amina	    |     8     |   	24	    |       3	       |            56	        |    9      |
Amanzi          |     Asmara	|     3     |   	49	    |       24	       |            20	        |    4      |
Amanzi          |     Bello	    |     3     |   	53	    |       20	       |            22	        |    3      |

**5.** In which province(s) do all towns have less than 50% access to home taps (including working and broken)?

````sql
SELECT
    SELECT
    *
FROM
    town_aggregated_water_access
WHERE
    (tap_in_home + tap_in_home_broken) >= 50;
````

**Results:**

province_name	|   town_name	|   river	|   shared_tap	|   tap_in_home    |    tap_in_home_broken  |   well    |
:--------------:|:-------------:|:---------:|:-------------:|:----------------:|:----------------------:|:---------:|
Amanzi          |     Amina     |     8     |       24      |	     3         |	        56          |	  9     |
Amanzi          |     Dahabu    |     3     |       37      |	     55        |	        1           |	  4     |
Akatsi          |     Harare    |     2     |       17      |	     28        |	        27          |	  27    |
Kilimani        |     Harare    |     7     |       11      |	     30        |	        20          |	  31    |
Sokoto          |     Ilanga    |     16    |       12      |	     36        |	        15          |	  21    |
Akatsi          |     Kintampo  |     2     |       15      |	     31        |	        26          |	  26    |
Akatsi          |     Lusaka    |     2     |       17      |	     28        |	        28          |	  26    |
Amanzi          |     Rural     |     3     |       27      |	     30        |	        30          |	  10    |
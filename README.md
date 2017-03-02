# NICAR17_Adv_SQL
[NICAR 2017 conference](https://ire.org/conferences/nicar2017/) tutorial that walks through truthing data, joins, updates and other concepts.

This class is based on similar classes taught by Liz Lucas at NICAR15 and Kendall Taggart at NICAR16. We're going to be using SQLite through a free Firefox plugin that makes it easier to use SQLite on your computer. It should already be on the classroom computers, but you can also download it [here](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/).

All the SQL for the class is in this file: SQL_queries.md

We're going to use inspection data from the Department of Labor/Occupational Safety and Health Administration. 

Here's the description of OSHA data from [DOL's website](http://ogesdw.dol.gov/views/data_summary.php): "The dataset consists of inspection case detail for approximately 100,000 OSHA inspections conducted annually. The dataset includes information regarding the impetus for conducting the inspection, and details on citations and penalty assessments resulting from violations of OSHA standards. Additionally, accident investigation information is provided, including textual descriptions of the accident, and details regarding the injuries and fatalities which occurred."

Inspection, accident and injury datasets were downloaded from the [DOL's site](http://ogesdw.dol.gov/views/data_summary.php) in February 2016. The files that are currently up on that site are incomplete and erroneous, and should not be used for analysis until they've been corrected by OSHA and thoroughly vetted for completeness (It's been a week and OSHA can't tell me when this problem will be corrected). OSHA also has a seemingly modern API, but as of late February 2017 the servers were down and no data could be accessed.


Record layouts:
* [Data dict from OSHA](http://enforcedata.dol.gov/views/data_dictionary.php)
Layouts for the tables we'll be using

What we'll cover:

* Truthing data tables and joins
* DISTINCT
* CREATE TABLE
* ALTER TABLE to add columns
* UPDATE to populate new columns
* Dealing with dates in SQLite (strftime)
* How to calculate difference in days between two dates
* Wildcards for more complex filtering
* Aliases in table names
* Subqueries

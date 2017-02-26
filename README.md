# NICAR17_Adv_SQL
NICAR 2017 tutorial walking through truthing data, joins, updates and other concepts

This class is based on similar classes taught by Liz Lucas at NICAR15 and Kendall Taggart at NICAR16. We're going to be using SQLite through a free Firefox plugin that makes it easier to use SQLite on your computer. It should already be on the classroom computers, but you can also download it here.

All the SQL for the class is in this file: SQL_queries.md

We're going to use inspection data from the Department of Labor/Occupational Safety and Health Administration.

Here's the description from DOL's website: "The dataset consists of inspection case detail for approximately 100,000 OSHA inspections conducted annually. The dataset includes information regarding the impetus for conducting the inspection, and details on citations and penalty assessments resulting from violations of OSHA standards. Additionally, accident investigation information is provided, including textual descriptions of the accident, and details regarding the injuries and fatalities which occurred."

Inspection, accident, accident_injury were downloaded from the DOL's site on 02/24/2017.

For this class, I limited the data to Jan 1, 2016 through February 24, 2017.

Record layouts:

Data dict from OSHA
Layouts for the tables we'll be using

What we'll cover:

Truthing data tables and joins
DISTINCT
CREATE TABLE
ALTER TABLE to add columns
UPDATE to populate new columns
Dealing with dates in SQLite (strftime)
How to calculate difference in days between two dates
Wildcards for more complex filtering
Aliases in table names
Subqueries

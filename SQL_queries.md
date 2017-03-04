# Below are the SQL commands used for Advanced SQL for analysis at NICAR16
* These commands were written in SQLite

## SET YOURSELF UP PROPERLY

Indexing is a way to help SQL comb through your data and find what it needs faster.  It's used in matching data between tables, so becomes important when you're performing joins. It might not make much difference in a smaller dataset like this, but when you're handling millions of records, having indexes on some key columns will help you immensely. (This is a really basic walkthrough of this concept; you'll want to read up to understand it better, and you should definitely read more before creating a clustered index)

Indexing nomenclature: You'll need to run a query to name each index you create. 
A good basic nomenclature for index naming is:
* PK_tablename_columnname on primary keys (these need to be unique, single values in your table. can't be null)
* IX_tablename_columnname on single-column indexes (these do not have to be unique values)
* CLIX_tablename_columnname on clustered indexes (these are indexes created across multiple fields, so an index that clusters city and state, for instance)

## KNOW YOUR DATA

How many records are there? 

    SELECT count(*) 
    FROM inspection;

Check for any duplicates — we know from the data dictionary 
  that 'activity_nr' should be the unique identifier for an 
  inspection. Let's check to see if the same 'activity_nr' 
  shows up more than once.

    SELECT activity_nr, count(*)
    FROM inspection
    GROUP BY activity_nr
    HAVING count(*) > 1;

Alternatively, we could count the number of distinct 'activity_nr' records:

    SELECT count(DISTINCT activity_nr)
    FROM inspection;

So activity_nr is definitely a unique value. Let's make it a primary key:

    CREATE INDEX PK_inspection_activity_nr ON inspection(activity_nr);
    
You'll run this and it'll look like nothing happened, but that's fine. You will see under "Indexes" now that one exists. While we're at it, let's also create indexes on the other two tables.

The three tables we're messing with are connected through these fields:  
    Inspection.activity_nr --> accident_injury.rel_insp_nr, accident_injury.summary_nr --> accident.summary_nr

On accident_injury, rel_insp_nr links that table to the related inspection -- but there can be more than one injury per inspection, so this isn't a unique value. So we'll just use a regular single-column index. Same goes for summary_nr in both tables - there can be more than one value present, so it's not a PK:

    CREATE INDEX IX_accident_injury_rel_insp_nr ON accident_injury(rel_insp_nr);
    CREATE INDEX IX_accident_injury_summary_nr ON accident_injury(summary_nr);
    CREATE INDEX IX_accident_summary_nr on accident(summary_nr);

OK, back to truthing your data. 
What's the date range? 

    SELECT min(open_date) as earliest, max(open_date) as latest
    FROM inspection;
    
Alternatively, we could find the 10 earliest and 10 latest dates:

    SELECT open_date
    FROM inspection
    ORDER BY 1
    LIMIT 10;

    SELECT open_date
    FROM inspection  
    ORDER BY 1 DESC
    LIMIT 10;

It's always good to check out a few other common fields to check how clean or dirty your data is. State names are always a good place to start when you're looking for missing or unusual values, so we'll check there. 

    SELECT site_state, count(*)
    FROM inspection
    GROUP BY site_state;
    
Overall looks pretty good — some blanks, some entries for the UK.
  We could look into the blanks to see if there's an obvious reason for them. 

    SELECT *
    FROM inspection   
    WHERE site_state IS NULL;
    
## Clean it up and prep for analysis
What if we want to take a closer look at inspections by year, for example? If you look at the columns included in the inspections table, there is open_date, but no open_year column... so let's make one.

But first: data types are the formats of your column -- for instance, zip codes should be text or varchar (string fields) while temperatures or total employees on location at the time of the accident you'd want to express as integers or numeric (number fields). Get to know your datatypes, because there's certain things you can't do with some (i.e., you can't add strings). Data dictionaries are good places to start when you're importing your data, because they'll help you set data types for each field correctly. For information on how SQLite handles data types, there's [this](http://www.sqlitetutorial.net/sqlite-data-types/).

When it comes to data types, dates are pretty much always hard. Like other programs, SQLite has a few quirks in dealing with dates. Here's the [documention](https://www.sqlite.org/lang_datefunc.html) for dealing with dates.


Let's make a column with just the year the inspection was opened. 

    ALTER TABLE inspection       
    ADD COLUMN open_year TEXT;

And then:

    SELECT open_date, strftime('%Y', open_date) as open_year    
    FROM inspection;

And then:

    UPDATE inspection    
    SET open_year = strftime('%Y', open_date);

How's it look?

    SELECT open_date, open_year       
    FROM inspection;       

How many inspections were there per year?

    SELECT open_year, count(*) as total_inspections
    FROM inspection     
    GROUP BY open_year
    ORDER BY total_inspections DESC;     

Check your work

    SELECT substr(open_date,1,4), count(*) as total       
    FROM inspection  
    GROUP BY 1      
    ORDER BY 2 DESC;

Want to know which month inspections are opened?

    SELECT strftime('%m', open_date) as open_month, count(*)  
    FROM inspection
    GROUP BY open_month;

Now, how long was the longest case open? 

    SELECT activity_nr, open_date, close_case_date, (julianday(close_case_date) - julianday(open_date)) AS datediff
    FROM inspection
    ORDER BY datediff DESC;

Translate to years (approximately)

    SELECT activity_nr, open_date, close_case_date,  
    	(julianday(close_case_date) - julianday(open_date))/365 AS datediff 
    FROM inspection
    ORDER BY datediff DESC;
 

Let's pull out inspections relating to the United States Postal Service, just to flex our filter muscles:
Take a look at some of the different ways it's spelled.

    SELECT estab_name, count(*)
    FROM inspection
    GROUP BY estab_name
    ORDER BY 2 DESC;

This should catch most of them.

    SELECT estab_name, count(*)
    FROM inspection
    WHERE 
        estab_name LIKE '%u%s%postal%service%' OR
        estab_name LIKE '%USPS%'
    GROUP BY estab_name
    ORDER BY 2 DESC;
    
What if we want to get a count of the number of inspections for each establishment?

    SELECT estab_name, count(DISTINCT activity_nr)
    FROM inspection
    GROUP BY 1;
    
Let's take a look at how to use subqueries. What if I want to know how many of the total inspections were initiated because of a complaint or an accident? Take a look at the insp_type field. This query will get us part of the way.

    SELECT estab_name, COUNT(insp_type) AS acc_or_complaint_insp
    FROM inspection
    WHERE insp_type = 'A' OR insp_type = 'B'
    GROUP BY 1;

Then let's combine it with another query to be able to compare it to the total inspections:

    SELECT a.estab_name, count(DISTINCT activity_nr) AS totalinspections, b.acc_or_complaint_insp
    FROM inspection A
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS acc_or_complaint_insp
        FROM inspection
        WHERE insp_type = 'A' OR insp_type = 'B'
        GROUP BY 1) as B
    ON a.estab_name = b.estab_name
    GROUP BY 1;

How often was advanced notice given?

    SELECT adv_notice, count(*)
    FROM inspection
    GROUP BY 1;

It's fair to say that advanced notice doesn't seem to be the norm, but what if it is for one company?
Let's do another subquery to compare this to the total number of inspections.

    SELECT a.estab_name, count(DISTINCT activity_nr) AS totalinspections, b.advanced_notice_insp
    FROM inspection A
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS advanced_notice_insp
        FROM inspection
        WHERE adv_notice = 'Y'
        GROUP BY 1) as B
    ON a.estab_name = b.estab_name
    GROUP BY 1;

We can make that into a separate table to query: 

    CREATE TABLE advanced_notice_inspections AS
    SELECT a.estab_name AS establishment_name, count(DISTINCT activity_nr) AS totalinspections, 
        b.advanced_notice_insp AS advanced_notice_count
    FROM inspection A
    JOIN
        (SELECT estab_name, COUNT(insp_type) AS advanced_notice_insp
        FROM inspection
        WHERE adv_notice = 'Y'
        GROUP BY 1) as B
    ON a.estab_name = b.estab_name
    GROUP BY 1;

And then take a look at the results (what percent of inspections were announced in advanced? I filtered for places that had
	had more than 5 inspections)

    SELECT establishment_name, (advanced_notice_count*1.0)/(totalinspections*1.0) as pct_adv_notice
    FROM advanced_notice_inspections
    WHERE totalinspections > 5
    GROUP BY 1

    
Now we'll take a look at the inspections that are connected with accidents. 

How many inspections included an accident that involved an injury?

    SELECT count(*)
    FROM inspection A
    INNER JOIN accident_injury B
    ON A.activity_nr =  B.rel_insp_nr;
    
Let's check to make sure that every record in the accident_injury table match a record in the inspection table.

    SELECT *
    FROM accident_injury A
    LEFT JOIN inspection B
    ON A.rel_insp_nr = B.activity_nr
    WHERE B.activity_nr IS NULL;
    
Let's make a table with the result so we can query it later

    CREATE TABLE inspection_plus_injury AS     
    SELECT *    
    FROM inspection A
    INNER JOIN accident_injury B
    ON A.activity_nr =  B.rel_insp_nr;
    
Now let's join this with data from the accident table, which includes fields like:
	1) 'event_desc': Short description of event and 2) 'event_date'

    CREATE TABLE osha_analysis_master AS
    SELECT * 
    FROM inspection_plus_injury A
    INNER JOIN accident B     
    ON A.summary_nr = B.summary_nr;

And now let's take a peak.

    SELECT estab_name, event_desc
    FROM osha_analysis_master
    WHERE event_desc LIKE '%horse%';

Or...

    SELECT estab_name, event_keyword
    FROM osha_analysis_master
    WHERE event_keyword LIKE '%burn%';

 

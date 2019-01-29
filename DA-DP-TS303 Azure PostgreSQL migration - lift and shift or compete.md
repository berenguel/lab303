# Getting Started with Oracle to Azure PostgreSQL migrations (Compete) #
---

**Introduction**

In this lab you will start with logging into the lab environment. Once this is done, we are going to go through three exercises that combined, will deploy end-to-end migration. And in the end, use cases discussions will take place.
* Exercise 1: Assess the source Oracle database using ora2pg

* Exercise 2: Migrate an Oracle database to Azure Database for PostgreSQL

* Exercise 3: Use cases discussion 

  

**Estimated Time**

60 minutes



**Objectives**

At the end of this lab, you will be able to:

* *Demonstrate to customers how to deploy an assessment and migration from Oracle to PostgreSQL*

* *Deliver the approach for Azure PostgreSQL migrations to customers*

* *Solve typical use cases for migration to Azure PostgreSQL following the appropriated Microsoft approach and tooling*

  

**Logging into the lab environment **

* Log into the lab using your **corporate credentials**

* Go to the top left hand side lightning and select "ctrl+alt+del"

* Go to the top left hand side lightning again and select "Type Text" -> "Type Password"

  


###  Exercise 1: Assess an Oracle source database ###

The steps in this exercise will demonstrate how to assess an Oracle database to understand cost and effort of an Azure Database for PostgreSQL migration. The attendees are going to execute the assessment using **ora2pg** (tool) via *cmd* and analyze the result of the report.

This exercise should take no longer than 5-7 min.

First, let’s get familiar with the ora2pg tool:

1. ora2pg is installed in your c:\ directory

   ![](https://github.com/berenguel/lab303/blob/master/IMG/c_dir.jpg)

   

2. Navigate to c:\ora2pg and right click on ora2pg_dist.conf and select “edit with Notepad++”

   IMAGE HERE

3. Right-click to open the file ora2pg_hr.conf. This is the configuration file that should be used for this
   lab.

   IMAGE HERE

   Second, let’s create the migration project structure:

   1. Open windows command prompt or “**cmd**”. Go to Windows Menu on the bottom left-hand side of your screen and type “**cmd**” or go to search and type “**cmd**”.

   2. Navigate to the ora2pg folder by typing:

~~~
      c:\ora2pg
~~~

   3. Run the command below:

~~~
      ora2pg --project_base c:\ts303 -c ora2pg_hr.conf –init_project hr_migration
~~~


   ​	The output should be as following:

   ​	IMAGE HERE



4. Using the GUI, check that the folder structure was created properly:

   

   IMAGE HERE

   

Third, let’s run the assessment:

1. Go back to the “**cmd**” window. In case you closed: Open windows command prompt or “**cmd**”. Go to Windows Menu on the bottom left-hand side of your screen and type “**cmd**” or go to search and type “**cmd**”.

2. Navigate to the hr_migration project folder by typing:
~~~
cd c:\ts303\hr_migration
~~~

3. Run the following set of commands:

~~~
ora2pg -t SHOW_TABLE -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\tables.txt
ora2pg -t SHOW_COLUMN -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\columns.txt
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report.html
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf –-cost_unit_value 10 --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report2.html 
~~~



IMAGE HERE



4. Check the report results on the migration project folder by navigating to c:\ts303\hr_migration\reports\ and double click on report.html to open it in the browser:

   IMAGE HERE


~~~
Migration levels:

        A - Migration that might be run automatically

        B - Migration with code rewrite and a human-days cost up to 5 days

        C - Migration with code rewrite and a human-days cost above 5 days

    Technical levels:

        1 = trivial: no stored functions and no triggers

        2 = easy: no stored functions but with triggers, no manual rewriting

        3 = simple: stored functions and/or triggers, no manual rewriting

        4 = manual: no stored functions but with triggers or views with code rewriting

        5 = difficult: stored functions and/or triggers with code rewriting
~~~



**Summary:** In this exercise, you learnt how to assess an Oracle database and understand what the complexity is for the migration to Azure Database for PostgreSQL. You also learnt this is step 1 of end-to-end migration project.

### Exercise 2: Migrate an Oracle source database to Azure Database for PostgreSQL ###

The steps in this exercise covers how to migrate an Oracle database to Azure Database for PostgreSQL. The attendees are going to export the Oracle database data and code using ora2pg, fix some common migration problems and import the fixed data, ddls and code into Azure Database for PostgreSQL. This exercise should take no longer than 30 min.

1. Export all the objects categories by running the following commands on cmd:

~~~
   ora2pg -t DBLINK -p -o dblink.sql -b %namespace%/schema/dblinks -c %namespace%/config/ora2pg.conf

   ora2pg -t DIRECTORY -p -o directory.sql -b %namespace%/schema/directories -c %namespace%/config/ora2pg.conf

   ora2pg -p -t FUNCTION -o functions2.sql -b %namespace%/schema/functions -c %namespace%/config/ora2pg.conf

   ora2pg -t GRANT -o grants.sql -b %namespace%/schema/grants -c %namespace%/config/ora2pg.conf

   ora2pg -t MVIEW -o mview.sql -b %namespace%/schema/mviews -c %namespace%/config/ora2pg.conf

   ora2pg -p -t PACKAGE -o packages.sql -b %namespace%/schema/packages -c %namespace%/config/ora2pg.conf

   ora2pg -p -t PARTITION -o partitions.sql -b %namespace%/schema/partitions -c %namespace%/config/ora2pg.conf

   ora2pg -p -t PROCEDURE -o procs.sql -b %namespace%/schema/procedures -c %namespace%/config/ora2pg.conf

   ora2pg -t SEQUENCE -o sequences.sql -b %namespace%/schema/sequences -c %namespace%/config/ora2pg.conf

   ora2pg -p -t SYNONYM -o synonym.sql -b %namespace%/schema/synonyms -c %namespace%/config/ora2pg.conf

   ora2pg -t TABLE -o table.sql -b %namespace%/schema/tables -c %namespace%/config/ora2pg.conf

   ora2pg -t TABLESPACE -o tablespaces.sql -b %namespace%/schema/tablespaces -c %namespace%/config/ora2pg.conf

   ora2pg -p -t TRIGGER -o triggers.sql -b %namespace%/schema/triggers -c %namespace%/config/ora2pg.conf

   ora2pg -p -t TYPE -o types.sql -b %namespace%/schema/types -c %namespace%/config/ora2pg.conf

   ora2pg -p -t VIEW -o views.sql -b %namespace%/schema/views -c %namespace%/config/ora2pg.conf
~~~


   2. Finally export the data, by running the following command on cmd
~~~
   ora2pg -t COPY -o data.sql -b %namespace/data -c %namespace/config/ora2pg.conf
~~~


   3. Perform the manual fixes as following:

   a) DataType change

   b) Synonym

  4. Prepare the Azure Database for PosgreSQL

  5. Run all files against the Azure Database for PostgreSQL server


~~~
   psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -L %namespace%\ schema\sequences\create_sequences.log

   psql -f %namespace%\schema\tables\table.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -L %namespace%\schema\tables\create_table.log

psql -f %namespace%\data\table1.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -L %namespace%\data\table1.log
~~~

During the compilation of files, check the logs and correct the necessary syntaxes that ora2pg was unable to convert out of the box.

   6..       Run Migration validation
~~~
   ora2pg -t TEST -c config/ora2pg.conf > migration_diff.txt
~~~

**Summary:** In this exercise, you learnt how to deploy an end-to-end migration from Oracle to Azure Database for PostgreSQL. In addition to that, you learnt the approach for this kind of migrations and how they can quickly executed for customers

  

   ### Exercise 3: Use Case and Pattern Discussion ###

   In this exercise, you will understand what the use cases for this migration.

   1.       Archiving

   2.       Single Tenant

   3.       MultiTenant

**Summary**: In this exercise, you learnt how the simple steps deployed in this lab can easily help customers on migrating from Oracle to Azure Database for PostgreSQL
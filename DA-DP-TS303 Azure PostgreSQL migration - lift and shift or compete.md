Getting Started with Oracle to Azure PostgreSQL migrations (Compete) 
---

**Introduction**

In this lab you will start with logging into the lab environment. Once this is done, we are going to go through three exercises that combined, will deploy end-to-end migration. And in the end, use cases discussions will take place.
* Exercise 1: Assess the source Oracle database using ora2pg

* Exercise 2: Migrate an Oracle database to Azure Database for PostgreSQL

* Exercise 3: Test the migration 

  

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

1. First, let’s understand **ora2pg** how ora2pg works. **ora2pg** is installed in your C:\ directory :

    **ora2pg ** relies on its powerful APIs which you can invoke via command line. The commands that you execute are going to be depending on what is written in the *default configuration file*, unless you tell otherwise. The *default config file* is called ora2pg_dist.conf and is under your **C:\ora2pg** installation:

- [x] Task:
   - Navigate to C:\ora2pg using via Windows folders (UI)
   - Open both ora2pg_dist.conf and ora2pg_hr.conf with Notepad++   -> right click on the file and select "Edit with Notepad++"

2. Now that you are familiar with how **ora2pg** works,  let’s create the migration project structure, as this is the easiest way of organizing the migration process:

 - [x] Task:
     - On Windows Command Prompt or “**cmd**”,  navigate to ora2pg folder
     - Run the command to create a migration template

        
	~~~
	cd c:\ora2pg
	~~~
	~~~
	ora2pg --project_base c:\ts303 -c ora2pg_hr.conf –-init_project 		hr_migration
	~~~

3. Let’s run the assessment:

  - [x] Task:

     - On Windows Command Prompt or "**cmd**", navigate to the hr_migration folder
     - Run the following sequence of assessment commands

~~~
cd c:\ts303\hr_migration
~~~

~~~
ora2pg -t SHOW_TABLE -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\tables.txt
~~~
~~~
ora2pg -t SHOW_COLUMN -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\columns.txt
~~~
~~~
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report.html
~~~
~~~
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf –-cost_unit_value 10 --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report2.html 
~~~



4. Check the report results:

- [x] Task:

   - Navigate to c:\ts303\hr_migration\reports\  via Windows folders (UI)

   - Double click on report.html to open it in the browser (Internet Explorer)

     

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



**Summary:** In this exercise, you learnt how to assess an Oracle database and understand what the complexity is for the migration to Azure Database for PostgreSQL. You also learnt this is step 1 of an end-to-end migration project.



### Exercise 2: Migrate an Oracle source database to Azure Database for PostgreSQL ###

The steps in this exercise will demonstrate how to migrate an Oracle database to Azure Database for PostgreSQL. The attendees are going to export the Oracle database objects and code using ora2pg, fix some common migration problems and import the fixed objects and data into Azure Database for PostgreSQL. This exercise should take no longer than 30 min.

1. Let's setup the Azure Database for PostgreSQL

   - [x] Task:

   - Connect to the Azure Subscription via browser (Internet Explorer)

   - Click on "All resources" - the lab303pg Azure Database for PostgreSQL is already set

   - Navigate to the "Connection Security" blade make sure the "Firewall rules" are properly set, as following:

     ​	Allow access to Azure services ON

     ​	Enforce SSL Connection DISABLED

   - Click Save

     

2. Let's connect to pgAdmin4 via browser and work on the database settings:

   - [x] Task:

   - Go to the "Windows" button on the bottom left-hand side of your screen
   - Click on pgAdmin 4 v4 (look for an elephant)
   - The connection with the PostgreSQL Server should be already set, so click on it to expand and connect
   - Expand the "Databases(3)" blade and right-click on "postgres" and then "Query Tool"
   - Run the following command:

   ```
   --@postgresql
   CREATE DATABASE lab303;
   CREATE ROLE HR LOGIN PASSWORD 'test303';
   GRANT CONNECT ON DATABASE lab303 TO HR;
   GRANT ALL PRIVILEGES ON DATABASE lab303 TO HR;
   
   ```

   - Expand the "Databases(4)" blade and right-click on "lab303" and then "Query Tool"
   - Run the following commands:

   ~~~
   --@lab303
   CREATE SCHEMA HR;
   GRANT USAGE ON SCHEMA HR TO HR;
   GRANT HR TO pgadmin;
   ALTER SCHEMA HR OWNER TO HR;
   ~~~

After configuring the Azure Database for PostgreSQL equivalent of the Oracle environment, lets export the Oracle schema.

3. Oracle database Objects export:

   - [x] Task:

   - Go back to **cmd**. 
   - Export all the objects categories by running the following commands:

~~~
cd c:\ora2pg
~~~
~~~
ora2pg -p -t PROCEDURE -o procs.sql -b C:\ts303\hr_migration\schema\procedures -c C:\ora2pg\ora2pg_hr.conf
~~~
~~~
ora2pg -t SEQUENCE -o sequences.sql -b C:\ts303\hr_migration\schema\sequences -c C:\ora2pg\ora2pg_hr.conf
~~~
~~~
ora2pg -t TABLE -o table.sql -b C:\ts303\hr_migration\schema\tables -c C:\ora2pg\ora2pg_hr.conf
~~~
~~~
ora2pg -p -t TRIGGER -o triggers.sql -b C:\ts303\hr_migration\schema\triggers -c C:\ora2pg\ora2pg_hr.conf
~~~
~~~
ora2pg -p -t VIEW -o views.sql -b C:\ts303\hr_migration\schema\views -c C:\ora2pg\ora2pg_hr.conf
~~~

4. Oracle Data export:
   - [x] Task:

~~~
ora2pg -t COPY -o data.sql -b C:\ts303\hr_migration\data -c C:\ora2pg\ora2pg_hr.conf
~~~



5. Run files against the Azure Database for PostgreSQL server, to figure out the errors (password: test303)

   - [x] Task:


~~~
cd C:\Program Files (x86)\pgAdmin 4\v4\runtime
~~~
~~~
psql -f C:\ts303\hr_migration\schema\tables\table.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\schema\tables\create_table.log
~~~
~~~
psql -f C:\ts303\hr_migration\schema\sequences\sequences.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\schema\sequences\create_sequences.log
~~~
~~~
psql -f C:\ts303\hr_migration\schema\procedures\procs.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\schema\procedures\create_procedure.log
~~~
~~~
psql -f C:\ts303\hr_migration\schema\triggers\triggers.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\schema\triggers\create_trigger.log
~~~
~~~
psql -f C:\ts303\hr_migration\schema\views\views.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\schema\views\create_view.log
~~~
~~~
psql -f C:\ts303\hr_migration\data\data.sql -h lab303pg.postgres.database.azure.com -p 5432 -U hr@lab303pg -d lab303 -L C:\ts303\hr_migration\data\data_import.log 
~~~

Typical Oracle to Azure Database for PostgreSQL fixes:

8. [https://microsoft.sharepoint.com/:w:/r/teams/sqlaa/jumpstart/_layouts/15/Doc.aspx?sourcedoc=%7B18D0D9BE-8303-4791-83DA-B02591AD261B%7D&file=Oracle%20to%20Azure%20PostgreSQL%20Migration%20Workarounds%20v1.0.docx&action=default&mobileredirect=true](Link here)

**Summary:** In this exercise, you learnt how to deploy an end-to-end migration from Oracle to Azure Database for PostgreSQL. In addition to that, you learnt the approach for this kind of migrations and how they can quickly executed for customers.

  

   ### Exercise 3: Migration Testing ###

      1.       Run Migration validation

```
   ora2pg -t TEST -c C:\ora2pg\ora2pg_hr.conf > migration_diff.txt
```



**Summary**: In this exercise, you learnt how to easily test the migration from Oracle to Azure Database for PostgreSQL.


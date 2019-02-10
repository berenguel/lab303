Getting Started with Oracle to Azure PostgreSQL migrations (Compete) 
---

**Introduction**

In this lab you will start with logging into the lab environment. Once this is done, we are going to go through three exercises that combined, will deploy end-to-end migration. And in the end, use cases discussions will take place.
* Exercise 1: Assess an Oracle schema using ora2pg

* Exercise 2: Migrate an Oracle schema to Azure Database for PostgreSQL

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

* Go to the top left-hand side "**Thunderbolt**" and select "**Ctrl+Alt+Del**"

* Go to the top left-hand side "**Thunderbolt**" again and select "**Type Text**" -> "**Type Password**"

  


###  Exercise 1: Assess an Oracle source database ###

The steps in this exercise will demonstrate how to assess an Oracle database to understand cost and effort of an Azure Database for PostgreSQL migration. You are going to execute the assessment using **ora2pg** (tool) via *cmd* and analyze the result of the report.

This exercise should take no longer than 5 min.

1. Let's run an assessment


 - [x] Task:
     - On “**cmd**”,  navigate to ora2pg folder
     - Run the command to create a migration template

        
     ~~~
     cd c:\ora2pg
     ~~~
     ~~~
     ora2pg --project_base c:\ts303 -c ora2pg_hr.conf –-init_project 		hr_migration
     ~~~

2. Run the assessment:

  - [x] Task:

     - On "**cmd**", navigate to the hr_migration folder
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



3. Check the report results:

- [x] Task:

   - Navigate to **c:\ts303\hr_migration\reports\**  via UI

   - Double click on **report.html** to open the report in your browser

     

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



**Summary:** In this exercise, you learnt how to assess an Oracle schema and understood how to articulate the migration complexity of a migration to Azure Database for PostgreSQL.



### Exercise 2: Migrate an Oracle schema to Azure Database for PostgreSQL ###

The steps in this exercise will demonstrate how to migrate an Oracle database to Azure Database for PostgreSQL. The attendees are going to export the Oracle database objects and code using ora2pg, fix some common migration problems and import the fixed objects and data into Azure Database for PostgreSQL. This exercise should take no longer than 25 min.

1. Let's setup the Azure Database for PostgreSQL

   - [x] Task:
   - Go to portal.azure.com and pick your Microsoft account
   - Click on "**Create a Resource**" and type "**Azure Database for PostgreSQL**"

   ![1549831354860](C:\Users\pabereng\AppData\Roaming\Typora\typora-user-images\1549831354860.png)

   

   - Select "**Azure Database for PostgreSQL**" and click on "Create"

   - Fill the required information as following:

     Server Name: **lab303pg**

     Resource Group: **lab303rg**

     Select Source: **Blank**

     Server admin login name: **pgadmin**

     Password: **LabAdmin303**

     Location: (whatever suits you)

     Version :**10** 

     Pricing Tier: **General Purpose 4 vCore(s)** - default

   

   ![1549833666701](C:\Users\pabereng\AppData\Roaming\Typora\typora-user-images\1549833666701.png)

   

   - Click "**Create**"

     


2. Oracle database Objects export:

- [x] Task:

- Go back to Windows **cmd** in your lab environment
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



3. Oracle Data export:

- [x] Task:

~~~
ora2pg -t COPY -o data.sql -b C:\ts303\hr_migration\data -c C:\ora2pg\ora2pg_hr.conf
~~~



4. Configuring the target  "Azure Database for PostgreSQL"  in your Azure Subscription:

- [x] Task:

- Go to your Azure Subscription and find your Azure Database for PostgreSQL resource. One of the ways of doing this is by doing the following:

- Click on "**Go to resource**"

![1549832468425](C:\Users\pabereng\AppData\Roaming\Typora\typora-user-images\1549832468425.png)

- Go to the "**Connection Security**" blade make sure the "**Firewall rules**" are properly set, as following:

  ​	Add a new rule: Rule Name: new_rule, Start IP: 1.1.1.1, End IP: 255.255.255.255

  ​	Enforce SSL Connection DISABLED

- Click "Save"

  

  ![1549832443872](C:\Users\pabereng\AppData\Roaming\Typora\typora-user-images\1549832443872.png)

  

  

5. Let's create the Azure Database for PostgreSQL **lab303** database:

- [x] Task:

  Run the following commands to configure the target **lab303** PostgreSQL database:

```
cd C:\Program Files (x86)\pgAdmin 4\v4\Runtime
```
```
psql -h lab303pg.postgres.database.azure.com -p 5432 -U pgadmin@lab303pg -d postgres 
```

Enter the Password when prompted. The password is: **LabAdmin303**

```
CREATE DATABASE lab303;
CREATE ROLE HR LOGIN PASSWORD 'test303';
GRANT CONNECT ON DATABASE lab303 TO HR;
GRANT ALL PRIVILEGES ON DATABASE lab303 TO HR;

```

```
\c lab303
```

```
CREATE SCHEMA HR;
GRANT USAGE ON SCHEMA HR TO HR;
GRANT HR TO pgadmin;
ALTER SCHEMA HR OWNER TO HR;
```

After configuring the Azure Database for PostgreSQL equivalent of the Oracle environment, lets export the Oracle schema.

5. Run files against the Azure Database for PostgreSQL server, to figure out the errors (password: test303)

   - [x] Task:


~~~
\o 'C:/ts303/hr_migration/schema/tables/create_table.log'
~~~
~~~
\i 'C:/ts303/hr_migration/schema/tables/table.sql'
~~~
~~~
\o 'C:/ts303/hr_migration/schema/sequences/create_sequences.log'
~~~
~~~
\i 'C:/ts303/hr_migration/schema/tables/sequences.sql'
~~~
~~~
\o 'C:/ts303/hr_migration/schema/procedures/create_procs.log'
~~~
~~~
\i 'C:/ts303/hr_migration/schema/procedures/procs.sql'
~~~
~~~
\o 'C:/ts303/hr_migration/schema/triggers/create_triggers.log'
~~~
~~~
\i 'C:/ts303/hr_migration/schema/triggers/create_trigger.sql'
~~~
~~~
\o 'C:/ts303/hr_migration/schema/views/create_views.log'
~~~
~~~
\i 'C:ts303/hr_migration/schema/views/views.sql'
~~~



Typical Oracle to Azure Database for PostgreSQL fixes:

8. [https://microsoft.sharepoint.com/:w:/r/teams/sqlaa/jumpstart/_layouts/15/Doc.aspx?sourcedoc=%7B18D0D9BE-8303-4791-83DA-B02591AD261B%7D&file=Oracle%20to%20Azure%20PostgreSQL%20Migration%20Workarounds%20v1.0.docx&action=default&mobileredirect=true](Link here)

**Summary:** In this exercise, you learnt how to deploy the migration of data and schema from
Oracle to Azure Database for PostgreSQL. 

  

   ### Exercise 3: Migration Testing ###

      1.       Run Migration validation

```
   ora2pg -t TEST -c C:\ora2pg\ora2pg_hr.conf > migration_diff.txt
```



**Summary**: In this exercise, you learnt how to easily test the migration from Oracle to Azure Database for PostgreSQL using ora2pg.


# detector
What is detector? It's a YAML Configuration style for running data quality checks on a table. All you need to do is to define column names for the table and the common checks (trending, uniqueness, not-null, etc) and the Python framework will auto generate the SQL for running in database. Current support is for Snowflake and Presto.

# how it works
Using the YAML configuration like this format:

> target_table:
>     name: schema.table
>  
> notification_email: user@company.com
> enable_email: true
>  
> dq:
>     trending:
>         enabled: true
>         columns:
>           - count(*)
>         threshold: 0.3
>  
>     trending-2:
>         enabled: true
>         columns:
>           - sum(gsv)
>         threshold: '+0.1'
>  
>     up_to_date:
>         enabled: true
>         columns:
>           - updated_date
>  
>     day_to_day:
>         enabled: true
>         columns:
>           - count(*)
>         threshold: 0.3
>         group_by: dw_modified_ts
>         num_days: 7
>  
>     missing_dates:
>         enabled: true
>         columns:
>           - etl_bucket_tstmp
>           - report_date
>  
>     std_dev:
>         enabled: true
>         columns:
>           - count(*)
>         threshold: 1.5
>  
> dq_custom:
>     check1:
>       enabled: true
>       stop_on_failure: true
>       description: This is for unit testing custom check1
>       sql: |
>         select case when count(*) == 0 then 0 else 1 end result
>         ,count(*) as tgt_value, NULL as src_value
>         from (
>           select
>           skey,
>           id,
>           count(*) cnt
>           from
>           schema.table
>           group by
>           1,2
>           having count(*) > 1
>         ) subq
>  
>     check2:
>       enabled: true
>       stop_on_failure: true
>       description: This is for unit testing custom check2
>       sql_file: <file>

Detail for generic check
- Trending: For tracking and compare the metric column from previous run
- Up_to_date: To check if data is up to date
- Empty_null: To check for column(s) if empty or null
- Unique: For checking any duplicated records
- Compare_to_source: Compare with source table defined in the top of YAML file
- Day_to_day: For tracking and compare the metric column(s) for the last x days
- Missing_dates: To check if any missing dates from the max of the specific column in the last 90 days
- Std_dev: For tracking and compare standard deviation of metric column(s) for the last 8 weeks on the same day

# how to run
python3 run_detector.py --file <file> [--run_hour YYYY-MM-DD HH:00:00] [--dry_run] [--unit_test] [--setup] [--variable] [--variable] ...

--file / -f:	The YAML configuration file containing all the data quality checks
--run_hour / -r:	To use for grouping all the ETL batch hour or date
--dry_run / -d:	Print all SQLs without executing in the system
--unit_test / -u:	Run the test without running result to the result table
--setup / -s:	Create the data result table only
--setup_stddev / -t:	Insert initial trending data for running standard deviation the first time
--variable / -v:	A variable list for string substitution in target_table_name and custom SQL

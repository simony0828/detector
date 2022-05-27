# detector
What is detector? It's a YAML Configuration style for running data quality checks on a table. All you need to do is to define column names for the table and the common checks (trending, uniqueness, not-null, etc) and the Python framework will auto generate the SQL for running in database. Current support is for Snowflake and Presto.

{% docs __overview__ %}
# dbt ELT Pipeline on Airbnb data

Hello, my name is Sourabh Khot and welcome to my project documentation!

In this project, I have built a data pipeline to perform ELT to analyze Airbnb data using dbt integrated with Snowflake, Github and preset.io (for dashboarding).

Click on the blue 'lineage graph' icon at bottom right to see dependencies between the jobs.
This dbt project contains 3 sources, 1 seed, 8 models, 1 snapshot, 16 tests, 1 analysis and 1 exposure.
The project files are available on my [Github](https://github.com/sourabhkhot/dbt_ELT_Airbnb/tree/ELT-branch-1).


### Pipeline Overview
Here is a graph of the data flow I created:
![input schema](https://github.com/sourabhkhot/dbt_ELT_Airbnb/blob/dbt_documentation/DataPipelineFlow.png?raw=true)

### Data Sources

The Airbnb data is sourced from InsideAirbnb.com containing data of 3 entities for Berlin (property listings, hosts and reviews) and is loaded in Snowflake database.
Reviews are considered as facts, which are linked to dimensions, which are listings and hosts.

The database contain 17499 listings, 14111 hosts and 410285 reviews. Here is the schema:
![input schema](https://github.com/sourabhkhot/dbt_ELT_Airbnb/blob/dbt_documentation/input_schema.png?raw=true)

Full moon are hypothesized to affect sleep, which can impact reviews left on the next day after full moon. 
To test this, full moon dates data is sourced from Kaggle.com and loaded as seed through dbt with Jinja.

### Tasks in the Pipeline

##### Raw Layer
* Loaded raw_hosts, raw_listings, raw_reviews in data warehouse from the web.
* Monitoring has been set for raw_reviews, which gives warming if no fresh data after 12 hours and error if none after 48 hours.

##### Staging Layer
* Basic checks of renaming column names to unique and relevant ones across tables.
* Models src_hosts, src_listings, src_reviews are materialized as CTEs/ephemeral, since they are read only once.
* Created snapshot (using Jinja) of raw_listings to track slowly changing dimension based on timestamp.

##### Core Layer
* For hosts and listings, perform cleansing such as replacing host names of NULL with Anonymous, mininum_nights of 0 with 1 and transforming price from string to number by removing $.
* For reviews, generate an incremental surrogate key using dbt-utils package.
* Models dim_hosts_cleansed and dim_listings_cleansed are materialized as views, since they are not used often.
* Model fct_reviews model is materialized as incremental/table append, since it is a huge table with only incremental changes, configured to fail if schema changes.
* Set up tests (generic, singular, custom) to check that keys are unique and not null, minimum nights is positive,
accepted values are present in room_type and is_superhost, test relationship references between models, 
and that all review dates are after listing creation date (consistent_created_at.sql).

##### Mart Layer
* Denormalize and joined table of dim_hosts_cleansed and dim_listings_cleansed with basic checks like renaming of columns
and single last update date being most recent of the two models.
* Join table of fct_reviews and seed_fullmoon_dates, adding a boolean column of 
whether review was left one day after full moon or not.
* Models mart_listings_w_hosts and mart_fullmoon_reviews are materialized as table.

##### Analytics Layer
* Peformed reviews_vs_fullmoon analysis to check proportion of sentiments if it was full moon or not,
by compiling analyses in dbt and executing the parsed SQL code in Snowflake
* Created exposure 'Executive Dashboard' using Preset.io and configured dependencies in dbt to aid documentation.

### Dashboard

Here's a screenshot of the dashboard. The live version is available at [Present.io link (requires login)](https://5dfed17f.us1a.app.preset.io/superset/dashboard/8/?edit=true&native_filters_key=mzGSw7XDbZR8Ihj4xa8UNAL0NB6nYsXalCxqW6Ew6xCtRXR3h_6O0ARwlinnSyxV)

![input schema](https://github.com/sourabhkhot/dbt_ELT_Airbnb/blob/dbt_documentation/dashboard.jpg?raw=true)

### Thank you

Thank you for viewing my project! Please share your feedback and questions at khot.s@northeastern.edu - Sourabh Khot

{% enddocs %}
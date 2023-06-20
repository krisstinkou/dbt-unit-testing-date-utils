# DBT Unit Testing Date Utils

This dbt package contains date related macros to support unit testing that can be (re)used across dbt projects.

## Installation Instructions

Add the following to packages.yml

```yaml
packages:
  - git: "https://github.com/krisstinkou/dbt-unit-testing-date-utils"
    revision: 0.1.0
```

## Available macros

- **dbt_unit_testing_date_utils.n_days_ago** Macro used to get datetime `days` days ago from `now_posixtime`.
- **dbt_unit_testing_date_utils.n_days_from_now** Macro used to get datetime in `days` days from `now_posixtime`.
- **dbt_unit_testing_date_utils.to_epoch** Macro used to convert datetime to Epoch format (in milliseconds).
- **dbt_unit_testing_date_utils.to_posix** Macro used to convert datetime to POSIX format (in seconds).
- **dbt_unit_testing_date_utils.to_iso** Macro used to convert datetime to ISO format (with milliseconds by default).
- **dbt_unit_testing_date_utils.generate_n_days_ago_variables** Macro used to conveniently set date/datetime values in a test.
**Note:** You can set both the **type of time period** ('d_dt', 'd_date', 'd_timestamp', etc.), and **time duration** value based on predefined constants ('1', '10', '100', etc.). Full macro code is available [here](https://github.com/krisstinkou/dbt-unit-testing-date-utils/blob/master/macros/date_utils.sql).

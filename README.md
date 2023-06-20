# DBT Unit Testing Date Utils

This dbt package contains date related macros to support date and time based queries unit testing that can be (re)used across dbt projects.

## Installation Instructions

Add the following to packages.yml

```yaml
packages:
  - git: "https://github.com/krisstinkou/dbt-unit-testing-date-utils"
    revision: 0.1.0
```

## All available macros

- **dbt_unit_testing_date_utils.n_days_ago** Macro used to get datetime `days` days ago from `now_posixtime`.
- **dbt_unit_testing_date_utils.n_days_from_now** Macro used to get datetime in `days` days from `now_posixtime`.
- **dbt_unit_testing_date_utils.to_epoch** Macro used to convert datetime to Epoch format (in milliseconds).
- **dbt_unit_testing_date_utils.to_posix** Macro used to convert datetime to POSIX format (in seconds).
- **dbt_unit_testing_date_utils.to_iso** Macro used to convert datetime to ISO format (with milliseconds by default).
- **dbt_unit_testing_date_utils.generate_n_days_ago_variables** Macro used to conveniently set date/datetime values in a test.
**Note:** You can set both the **type of time period** ('d_dt', 'd_date', 'd_timestamp', etc.), and **time duration** value based on predefined constants ('1', '10', '100', etc.). Full macro code is available [here](https://github.com/krisstinkou/dbt-unit-testing-date-utils/blob/master/macros/date_utils.sql).

## Why do we need this package? 

Because of its main feature!

In data testing, it is often necessary to test data that changes over time, such as daily aggregates or time-based metrics. By using the `generate_n_days_ago_variables` macro, you can easily generate a set of variables that represent dates in the past and future, relative to the current (or preassigned) date. These variables can then be used in tests to set date/datetime values for the data being tested.

Overall, the `generate_n_days_ago_variables` macro is a convenient way to generate a set of variables for testing time-based data, and can save time and effort when writing tests.

## Parameters

- `now_posixtime` (optional): The Unix epoch timestamp for the current date (default: 0, which represents the current Unix epoch timestamp).

## Result of macro call

The result of calling the `generate_n_days_ago_variables` macro is a dictionary that contains variables for the current date and the previous/future `n` days, where `n` is a list of integers ranging from 0 to 1000.

The keys of the dictionary are strings that represent the variables, and the values are the corresponding date/datetime values or Unix epoch timestamps/microsecond timestamps.

With the default parameters, part of the result might look something like this:

```python
{
    '-1d_dt': datetime.datetime(2023, 6, 19, 23, 33, 36, 635043), 
    '+1d_date': '2023-06-21', 
    '-1d_timestamp': '2023-06-19 23:33:36', 
    '+1d_epoch': 1687390416635.043, 
    '-1d_posix': 1687217616.635043, 
    '+1d_micros': 1687390416635000, 
    '-1d_micros_from_timestamp': 1687217616000000, 
    # ... and so on for other variables
}
```

## Key datetime format

The general datetime variables format is as follows: `{sign}{days}{format}`

The `sign` indicates what time it is talking about: the future (`+`) or the past (`-`) relative to the `now_posixtime` or current datetime.

The `days` means how many days ago/in the future relative to the `now_posixtime`  or current datetime.

The `format` can take one of the following values:

- `d_dt` - datetime object.
- `d_date` - datetime in `YYYY-MM-DD` format.
- `d_timestamp` - datetime in `YYYY-MM-DD hh:mm:ss` format.
- `d_epoch` - datetime in POSIX timestamp format in milliseconds.
- `d_posix` - datetime in POSIX timestamp format in seconds.
- `d_micros` - datetime in microseconds, rounded down to the nearest millisecond.
- `d_micros_from_timestamp` - datetime in microseconds, rounded down to the nearest second.

## Example of usage

To use the macro in your tests, simply call it in your project SQL tests script:

```sql
{% set dt = dbt_unit_testing_date_utils.generate_n_days_ago_variables() %}
```

Or you can call it with your own parameters:

```sql
{% set now_posixtime = dbt_unit_testing_date_utils.to_posix('2022-01-01 12:00:00') %}
{% set dt = dbt_unit_testing_date_utils.generate_n_days_ago_variables(now_posixtime=now_posixtime) %}
```

This will generate the variables and store them in the `dt` dictionary. You can then use the variables in your SQL queries:

```sql
SELECT *
FROM my_table
WHERE date_column >= '{{ dt["-5d_date"] }}'
  AND date_column <= '{{ dt["-1d_date"] }}'
```

You can take a look on the result of `generate_n_days_ago_variables` macro call with the following piece of code:

```sql
{% set dt = generate_n_days_ago_variables() %}
{% do log(dt, True) %}
```

## License

You can find license for this package [here](https://github.com/krisstinkou/dbt-unit-testing-date-utils/blob/master/LICENSE)
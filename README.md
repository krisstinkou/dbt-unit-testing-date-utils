# DBT Unit Testing Date Utils

This dbt package contains date related macros to support unit tests writing for the data that contains a lot of date and time based logic. With these marcos you can generate datetime variables that can be used in your unit tests. This package can be (re)used across dbt projects.

## Getting started

### We have dbt model

We have the table with information about users: `user_id`, `username`, `email` and `updated_at` as date and time in timestamp format.

### We would like to write unit tests

We would like write tests for this model using [dbt_unit_testing](https://github.com/EqualExperts/dbt-unit-testing) framework. 

Now we can write unit tests for our project. For example, we would like to mock mentioned model with [mock_ref](https://github.com/EqualExperts/dbt-unit-testing/blob/3bfdb45b516ebc393e43edc027ab51cb9fe5530a/macros/mock_builders.sql#L1) macro.

```sql
{{ config(tags=['unit-test']) }}

{% call dbt_unit_testing.test ('user', 'test user data') %}
  {% call dbt_unit_testing.mock_ref('user') %}
    SELECT '1' AS user_id, 'test_username1' AS username, 'email1@email.com' AS email, 1651255712000 AS updated_at
    SELECT '2' AS user_id, 'test_username2' AS username, 'email2@email.com' AS email, 1650391712000 AS updated_at
    SELECT '3' AS user_id, 'test_username3' AS username, 'email3@email.com' AS email, 1649527712000 AS updated_at
  {% endcall %}
  -- other unit tests code --
{% endcall %}
```

### We see inconvenient date and time format

We can see raw date and time data. But we are unable to discern a specific time (such as day, month, or year) without converting the timestamp data using a specific tool.

This package includes a script that generates a set of variables with date and time objects in different formats.

```sql
{{ config(tags=['unit-test']) }}

{% call dbt_unit_testing.test ('user', 'test user data') %}
  {%- set dt = dbt_unit_testing_date_utils.generate_n_days_ago_variables() -%}
  {% call dbt_unit_testing.mock_ref('user') %}
    SELECT '1' AS user_id, 'test_username1' AS username, 'email1@email.com' AS email, '{{ dt["-10d_epoch"] | int }}' AS updated_at
    SELECT '2' AS user_id, 'test_username2' AS username, 'email2@email.com' AS email, '{{ dt["-20d_epoch"] | int }}' AS updated_at
    SELECT '3' AS user_id, 'test_username3' AS username, 'email3@email.com' AS email, '{{ dt["-30d_epoch"] | int }}' AS updated_at
  {% endcall %}
  -- other unit tests code --
{% endcall %}
```

As you can observe, it is not necessary to manually define datetime data. We can use prepared datetime variables, which is significantly more convenient and visually comprehensible. In this example we can see that the `updated_at` field has values "10 days ago", "20 days ago", "30 days ago".

### Mock datetime to generate

With every test run you will get a different generation result (time does not stand still). For your tests you can define macro to mock current posix timestamp. For example as follows:

```sql
{% macro mocked_timestamp_posix() %}
  -- posix_time: 2022-01-01 12:00:00 --
  {{ 1641038400 | int }}
{% endmacro %}
```

And then you can use macro with mocked date:

```sql
{%- set posix_time = mocked_timestamp_posix() | int -%}
{%- set dt = dbt_unit_testing_date_utils.generate_n_days_ago_variables(posix_time) -%}
```

Thanks to these simple manipulations, you will get the same generation result.

### Override current_datetime

Besides `mocked_timestamp_posix` mocking you can also override macro `current_timestamp()` with the condition of using it for tests. To understand if this macro is used in tests you can use `flags` feature. For example as follows:

```sql
{% macro current_timestamp() %}
  {# 1652119712 seconds, 1652119712000 milliseconds #}
  {% if flags.WHICH == "test" -%}
    CAST('2022-05-09 18:08:32.000 UTC' as timestamp)
  {%- else -%}
    CAST(current_timestamp as timestamp)
  {%- endif %}
{% endmacro %}
```

### Main feature

So why we need this `dbt_unit_testing_date_utils` package? Because of its main feature!

In data testing, it is often necessary to test data that changes over time, such as daily aggregates or time-based metrics. By using the `generate_n_days_ago_variables` macro, you can easily generate a set of variables that represent dates in the past and future, relative to the current (or preassigned) date. These variables can then be used in tests to set date/datetime values for the data being tested.

Overall, the `generate_n_days_ago_variables` macro is a convenient way to generate a set of variables for testing time-based data, and can save time and effort when writing tests.

## Prerequisites

This package was developed as the helping hand for the tests written using `dbt_unit_testing` package.

Original package: [https://github.com/EqualExperts/dbt-unit-testing](https://github.com/EqualExperts/dbt-unit-testing)

Author's fork: [https://github.com/SOVALINUX/dbt-unit-testing](https://github.com/SOVALINUX/dbt-unit-testing)

## Installation Instructions

Add the following to packages.yml

```yaml
packages:
  - git: "https://github.com/krisstinkou/dbt-unit-testing-date-utils"
    revision: 0.1.0
```

## All available macros

- `dbt_unit_testing_date_utils.n_days_ago` macro used to get datetime `days` days ago from `now_posixtime`.
- `dbt_unit_testing_date_utils.n_days_from_now` macro used to get datetime in `days` days from `now_posixtime`.
- `dbt_unit_testing_date_utils.to_epoch` macro used to convert datetime to Epoch format (in milliseconds).
- `dbt_unit_testing_date_utils.to_posix` macro used to convert datetime to POSIX format (in seconds).
- `dbt_unit_testing_date_utils.to_iso` macro used to convert datetime to ISO format (with milliseconds by default).
- `dbt_unit_testing_date_utils.generate_n_days_ago_variables` macro used to conveniently set date/datetime values in a test.
> **_Note:_** You can set both the **type of time period** ('d_dt', 'd_date', 'd_timestamp', etc.), and positive or zero **time duration** value based on predefined constants (0, 1, 2, 3, etc.). You can also change these predefined constants in the macro code. Full macro code is available [here](https://github.com/krisstinkou/dbt-unit-testing-date-utils/blob/master/macros/date_utils.sql).

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

## Other examples of usage

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

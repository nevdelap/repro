# Repro

This reproduces the problem reported
[here](https://github.com/snowflakedb/snowflake-connector-python/issues/2499) in
a minimalish way. Since I don't know what dbt is doing exactly I've not made it
as minimal as might be desirable. Hopefully if debugging in the Snowflake
connector is turned on it will be enough to point at the source of the problem.

1. Create a database.

    - `create database test;`
    - `grant ownership on database <whatever_database> to role <whatever_role>;`
    - `grant ownership on schema <whatever_database>.public to role <whatever_role>;`

2. Clone the repository and set up these env vars:

    - DBT_ACCOUNT
    - DBT_USER
    - DBT_KEY_FILE
    - DBT_KEY_PASSPHRASE
    - DBT_ROLE `<whatever_role>`
    - DBT_DATABASE - `<whatever_database>`
    - DBT_SCHEMA - `public`

3. `uv sync`
4. `export DBT_PROFILES_DIR="$(pwd)/.dbt"`
5. `uv run dbt clean`
6. `uv run dbt deps`
7. `uv run dbt build`

    ```plaintext
    09:15:15 nevsmachine ~/data/repro > uv run dbt build
    23:15:27  Running with dbt=1.10.9
    23:15:27  Registered adapter: snowflake=1.10.0
    23:15:27  Found 1 model, 940 macros
    23:15:27
    23:15:27  Concurrency: 32 threads (target='dev')
    23:15:27
    23:15:30  1 of 1 START sql view model public.bogus ....................................... [RUN]
    23:15:30  1 of 1 OK created sql view model public.bogus .................................. [SUCCESS 1 in 0.21s]
    23:15:30
    23:15:30  Finished running 1 view model in 0 hours 0 minutes and 2.87 seconds (2.87s).
    23:15:30
    23:15:30  Completed successfully
    23:15:30
    23:15:30  Done. PASS=1 WARN=0 ERROR=0 SKIP=0 NO-OP=0 TOTAL=1
    09:15:31 nevsmachine ~/data/repro >
    ```

8. In `pyproject.toml` change `"snowflake-connector-python==3.16",` to `"snowflake-connector-python==3.17",`.

```plaintext
09:17:22 nevsmachine ~/data/repro > uv sync && uv run dbt build
Resolved 76 packages in 24ms
Prepared 1 package in 832ms
Uninstalled 1 package in 8ms
Installed 1 package in 12ms
 - snowflake-connector-python==3.16.0
 + snowflake-connector-python==3.17.0
23:17:32  Running with dbt=1.10.9
23:17:33  Registered adapter: snowflake=1.10.0
23:17:33  Found 1 model, 940 macros
23:17:33
23:17:33  Concurrency: 32 threads (target='dev')
23:17:33
WARNING:snowflake.connector.vendored.urllib3.connectionpool:Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<snowflake.connector.vendored.urllib3.connection.HTTPConnection object at 0x722128ef2e90>: Failed to establish a new connection: [Errno -2] Name or service not known')': /computeMetadata/v1/instance/service-accounts/default/email
WARNING:snowflake.connector.vendored.urllib3.connectionpool:Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<snowflake.connector.vendored.urllib3.connection.HTTPConnection object at 0x722125a43820>: Failed to establish a new connection: [Errno -2] Name or service not known')': /
WARNING:snowflake.connector.vendored.urllib3.connectionpool:Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ConnectTimeoutError(<snowflake.connector.vendored.urllib3.connection.HTTPConnection object at 0x722126aff9d0>, 'Connection to 169.254.169.254 timed out. (connect timeout=0.2)')': /metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com
WARNING:snowflake.connector.vendored.urllib3.connectionpool:Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'ConnectTimeoutError(<snowflake.connector.vendored.urllib3.connection.HTTPConnection object at 0x722124679d30>, 'Connection to 169.254.169.254 timed out. (connect timeout=0.2)')': /metadata/instance?api-version=2021-02-01
23:17:35  1 of 1 START sql view model public.bogus ....................................... [RUN]
23:17:35  1 of 1 OK created sql view model public.bogus .................................. [SUCCESS 1 in 0.17s]
23:17:36
23:17:36  Finished running 1 view model in 0 hours 0 minutes and 2.69 seconds (2.69s).
23:17:36
23:17:36  Completed successfully
23:17:36
23:17:36  Done. PASS=1 WARN=0 ERROR=0 SKIP=0 NO-OP=0 TOTAL=1
09:17:37 nevsmachine ~/data/repro >
```

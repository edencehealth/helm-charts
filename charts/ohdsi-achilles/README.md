# edenceHealth OHDSI Helm Chart: Achilles

This is a repository for software related to the deployment of [Observational Health Data Sciences and Informatics](https://ohdsi.org/) (OHDSI) [software](https://github.com/OHDSI/) in a containerized environment.

This helm chart contains a deployment of [OHDSI Achilles](https://ohdsi.github.io/Achilles/)

## Deploying

When deploying this chart the following values will typically need site-specific configuration (for example in a `settings.yaml` file):

### `settings.yaml`

```yaml
achilles:
  pvcName: achilles-output
  dbSecretName: ohdsi-achilles-db-secret
  env:
    CDM_VERSION: "5.4"
    SOURCE_NAME: "ohdsi"
    CDM_SCHEMA: "omopcdm"
    VOCAB_SCHEMA: "omopcdm"
    DB_DBMS: "postgresql"
    DB_HOSTNAME: "127.0.0.1"
    DB_NAME: "omopcdm"
    DB_PORT: 5432
    JSON_EXPORT: "TRUE"
    DQD: "TRUE"
    DQD_VERBOSE: "TRUE"
    DQD_JSON_FILE: "TRUE"
    SKIP_ACHILLES: "FALSE"
    # DQD_FIELD_THRESHOLD_FILE: "/vocab_thresholds/OMOP_CDMv5.4_Field_Level.csv"
    # DQD_TABLE_THRESHOLD_FILE: "/vocab_thresholds/OMOP_CDMv5.4_Table_Level.csv"
```

### Database Secret

Create a database secret and (optionally) a migration-specific secret:

This is the kubernetes secret that holds DB connection username and password. The keys of this secret map to environment variables of the achilles container

secret key name | container environment variable
----------------|-------------------------------
username        | `DATASOURCE_USERNAME`
password        | `DATASOURCE_PASSWORD`

This command can be used to create the secret:

```sh
kubectl \
  create secret generic \
  ohdsi-k8s-db-secret \
  --from-literal=username='SomeUsernameYouSupply' \
  --from-literal=password='SomePasswordYouSupply'
```

### Configuration Information


#### General Options

Environment Variable   | Parsed Type     | Default    | Description
---------------------- | --------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------
`NUM_THREADS`          | integer         | `1`        | The number of threads to use when running achilles
`OPTIMIZE_ATLAS_CACHE` | boolean string  | `` (empty) | Enables the optimizeAtlasCache option to the achilles function
`SOURCE_NAME`          | string          | `NA`       | The source name used by the achilles function and included as part of the output path
`TIMESTAMP`            | string          | `AUTO`     | The timestamp-style string to use when calculating some file output paths. Defaults to a string derived from the current date & time.
`SKIP_ACHILLES`        | boolean string  | `` (empty) | This option prevents Achilles from running, which can be useful for running the other utilities like the DQD Shiny App
`OUTPUT_BASE`          | string          | `/output`  | The output path used by achilles
`S3_TARGET`            | string          | `` (empty) | Optional AWS S3 bucket path to sync with the output_base directory (for uploading results to S3)

#### CDM Options

Environment Variable   | Parsed Type      | Default    | Description
---------------------- | ---------------- | ---------- | ----------------------------------------
`CDM_VERSION`          | semantic version | `5`        | Which standard version of the CDM to use

#### Schema Options

Environment Variable | Parsed Type | Default      | Description
-------------------- | ----------- | ------------ | --------------------------------------------------------
`CDM_SCHEMA`         | string      | `public`     | DB schema name for CDM data [default: public]
`RESULTS_SCHEMA`     | string      | `results`    | DB schema name for results data [default: results]
`VOCAB_SCHEMA`       | string      | `vocabulary` | DB schema name for vocabulary data [default: vocabulary]

#### CDM DB Options

Environment Variable | Parsed Type | Default      | Description
-------------------- | ----------- | ------------ | ---------------------------------------------------
`DB_DBMS`            | string      | `postgresql` | The database management system for the CDM database
`DB_HOSTNAME`        | hostname    | `db`         | The hostname the database server is listening on
`DB_PORT`            | integer     | `5432`       | The port the database server is listening on
`DB_NAME`            | string      | `cdm`        | The name of the database on the database server

#### JSON Export Options

Environment Variable | Parsed Type    | Default    | Description
-------------------- | -------------- | ---------- | -------------------------------------------------
`JSON_EXPORT`        | boolean string | `` (empty) | Whether to run the Achilles exportToJson function
`JSON_COMPRESS`      | boolean string | `` (empty) | JSON files should be compressed into one zip file

#### DataQualityDashboard Options

Environment Variable         | Parsed Type    | Default             | Description
---------------------------- | -------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------
`DQD`                        | boolean string | false               | Whether to run the DataQualityDashboard functions
`DQD-SQL-ONLY`               | boolean string | false               | Return DQD queries but do not run them
`DQD-VERBOSE`                | boolean string | false               | Whether to write DataQualityDashboard info
`DQD-JSON-FILE`              | boolean string | false               | Whether to write a JSON file to disk
`DQD-SKIP-DB-WRITE`          | boolean string | false               | Skip writing results to the dqdashboard_results table in the results schema
`DQD-CHECK-NAMES`            | csv            | empty list          | Optional comma-separated list of check names to execute
`DQD-CHECK-LEVELS`           | csv            | TABLE,FIELD,CONCEPT | Comma-separated list of which DQ check levels to execute
`DQD-EXCLUDE-TABLES`         | csv            | empty list          | Comma-separated list of CDM tables to exclude from the checks
`DQD-TABLE-THRESHOLD-FILE`   | file path      | not set             | The optional location of the threshold file for evaluating the table checks; this is useful for overriding thresholds
`DQD-FIELD-THRESHOLD-FILE`   | file path      | not set             | The optional location of the threshold file for evaluating the field checks; this is useful for overriding thresholds
`DQD-CONCEPT-THRESHOLD-FILE` | file path      | not set             | The optional location of the threshold file for evaluating the concept checks; this is useful for overriding threshold

### Example Chart Deploymnet Commands

```sh
helm repo add edencehealth https://edencehealth.github.io/helm-charts
helm install -f settings.yml  edencehealth/ohdsi-achilles
```

to check for updates:

```sh
helm repo update edencehealth
```


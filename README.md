# Boxer Engineering - Migratex

Wrapper script to add support for DML database migrations to [migrate](https://github.com/golang-migrate/migrate).

We use the excellent [migrate](https://github.com/golang-migrate/migrate) tool for Google GCP Spanner database migrations however it does not support DML.
DML is useful when a new feauture requires some non-transactional data.
We work around this with a wrapper tool `migratex` which interleaves DML migrations with `migrate's` DDL migrations.

`migratex`uses the [Go Cloud Spanner client library](https://cloud.google.com/spanner/docs/reference/libraries#client-libraries-install-go) and so can cache the Spanner session and leverage things like batch DML processing.

`migratex` requires a naming convention to recognize migrations.
In the following examples `_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE]` can be anything, `[REVISION]` must be an integer, optionally prefixed by zeros, and `[ENV_ID]` is and environment ID for which the migrations should be applied.
The environment ID is passed to `migratex` at runtime:

    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].ddl.up.sql
    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].[ENV_ID].dml.sql
    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].[ENV_ID].[ENV_ID].dml.sql
    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].all.dml.sql

DML can contain tokens and if so the tokens will be resolved if a JSON token definition file exists.
JSON token definition files are optional but there can only be one per DML file:

    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].[ENV_ID].dml.json
    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].[ENV_ID].[ENV_ID].dml.json
    [REVISION]_[SOME_BUINSESS_DOMAIN]_[SOME_FEATURE].all.dml.json

Note that there can only be one DML file for a revision for each environment.
Note that DML migration revision history is maintained in the table `DataMigrations`.

## Usage

```shell
gcloud config configurations activate [CONFIGURATION_NAME]

gcloud spanner databases create [SPANNER_DATABASE_ID] --instance=[SPANNER_INSTANCE_ID]

# DDL only using 'migrate'
migrate -path . -database spanner://projects/[GCP_PROJECT_ID]/instances/[SPANNER_INSTANCE_ID]/databases/[SPANNER_DATABASE_ID] up

go get -u github.com/goboxer/public-migratex

# DDL and DML using 'migratex'
migratex -env_id=[ENV_ID] -gcp_project_id=[GCP_PROJECT_ID] -spanner_instance_id=[SPANNER_INSTANCE_ID] -spanner_database_id=[SPANNER_DATABASE_ID]
```

## Contributors

Contributors in alphabetical order are Leonie Farmer, Jordan Felix, Billy Fung, Cai Gwatkin, Xiaona Jia, Rick Lee, Ling Liang, Parker Lin, Bruno Monteiro, Tom Wang, Steven Zhang, Yixin Zhang et al.

# Documentation

## Tutorial

You can get quickly started with the tutorial in **[`Google Colaboratory`](https://colab.research.google.com/drive/1ByFx_NOEfTZeaJ1Wtw3UwTH3H3-Sye2O?usp=sharing)**.

## Installation

In the following we describe different ways in which you can install and use Morph-KGC.

### PyPI

**[PyPI](https://pypi.org/project/morph-kgc/)** is the fastest way to install Morph-KGC:
``` bash
pip install morph-kgc
```

Some data sources and output formats require additional dependencies. Check **[Advanced Setup](https://morph-kgc.readthedocs.io/en/latest/documentation/#advanced-setup)** for specific installation instructions or install all the dependencies:

``` bash
pip install morph-kgc[all]
```

We recommend to use **[virtual environments](https://docs.python.org/3/library/venv.html#)** to install Morph-KGC.

### From Source

You can also grab the latest source code from the **[GitHub repository](https://github.com/morph-kgc/morph-kgc)**:
``` bash
pip install git+https://github.com/morph-kgc/morph-kgc.git
```

## Usage

Morph-KGC uses an **[INI file](https://en.wikipedia.org/wiki/INI_file)** to configure the materialization process, see **[Configuration](https://morph-kgc.readthedocs.io/en/latest/documentation/#configuration)**.

### Command Line

Installing Morph-KGC puts the **`morph-kgc`** command on your path. To run the engine using the **command line** you just need to execute the following:

``` bash
morph-kgc path/to/config.ini
```

Running the package as a module is equivalent, and is what the [Docker](https://morph-kgc.readthedocs.io/en/latest/documentation/#docker) image does:

``` bash
python -m morph_kgc path/to/config.ini
```

### Library

Morph-KGC can be used as a **library**, providing different methods to materialize the knowledge graph. It integrates with **[RDFLib](https://rdflib.readthedocs.io/en/stable/)** and **[Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)** to easily create and work with knowledge graphs in **[Python](https://www.python.org/)**.

The methods in the **API** accept the configuration as the **path to an INI file**, as an **INI string**, or as a **Python dictionary**.

``` python
import morph_kgc

# the path to an INI file
config = '/path/to/config.ini'

# an INI string
config = """
            [DataSource1]
            mappings: /path/to/mapping/mapping_file.rml.ttl
            db_url: mysql+pymysql://user:password@localhost:3306/db_name
         """

# a dictionary with one section per data source
config = {
    'CONFIGURATION': {'output_format': 'N-QUADS'},
    'DataSource1': {
        'mappings': '/path/to/mapping/mapping_file.rml.ttl',
        'db_url': 'mysql+pymysql://user:password@localhost:3306/db_name'
    }
}

# a flat dictionary, when there is a single data source
config = {
    'output_format': 'N-QUADS',
    'mappings': '/path/to/mapping/mapping_file.rml.ttl',
    'db_url': 'mysql+pymysql://user:password@localhost:3306/db_name'
}
```

#### [RDFLib](https://rdflib.readthedocs.io/en/stable/)

**`morph_kgc.materialize(config)`**

Materialize the knowledge graph to **[RDFLib](https://rdflib.readthedocs.io/en/stable/)**.

``` python
# generate the triples and load them to an RDFLib graph

graph = morph_kgc.materialize(config)
# or
graph = morph_kgc.materialize('/path/to/config.ini')

# work with the RDFLib graph
q_res = graph.query(' SELECT DISTINCT ?classes WHERE { ?s a ?classes } ')
```

{==

*__Note:__ [RDFLib](https://rdflib.readthedocs.io/en/stable/) does not read [RDF 1.2](https://www.w3.org/TR/rdf12-concepts/) triple terms or directional language-tagged strings, hence `materialize` does not support the [RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification) constructs that generate them. Use `materialize_set` or write the knowledge graph to a file instead.*

==}

#### [Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)

**`morph_kgc.materialize_oxigraph(config)`**

Materialize the knowledge graph to **[Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)**.

``` python
# generate the triples and load them to Oxigraph

graph = morph_kgc.materialize_oxigraph(config)
# or
graph = morph_kgc.materialize_oxigraph('/path/to/config.ini')

# work with Oxigraph
q_res = graph.query(' SELECT DISTINCT ?classes WHERE { ?s a ?classes } ')
```

#### Set of Triples

**`morph_kgc.materialize_set(config)`**

Materialize the knowledge graph to a Python **Set of triples**. Each element is a serialized statement without its trailing ` .`, so this is the method to use for knowledge graphs that no RDF library reads back yet, such as those with [RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification) triple terms.

``` python
# create a Python Set with the triples

graph = morph_kgc.materialize_set(config)
# or
graph = morph_kgc.materialize_set('/path/to/config.ini')

# work with the Python set
print(len(graph))
```

## Configuration

The configuration of Morph-KGC is done via an **[INI file](https://en.wikipedia.org/wiki/INI_file)**. This configuration file can contain the following sections:

**`{++CONFIGURATION++}`**

- Contains the parameters that **tune** the execution of Morph-KGC, see **[Engine Configuration](https://morph-kgc.readthedocs.io/en/latest/documentation/#engine-configuration)**.

**One section for each `{++DATA SOURCE++}`**

- Each input data source has its own section, see **[Data Sources](https://morph-kgc.readthedocs.io/en/latest/documentation/#data-sources)**.

**One `{++RESOURCE:<name>++}` section for each accessed resource**

- Resources are not materialized, they are accessed while materializing: a vocabulary to reconcile against, a SPARQL endpoint to query. See **[Resources](https://morph-kgc.readthedocs.io/en/latest/documentation/#resources)**.

**`{++DEFAULT++}`**

- It is **optional** and it declares variables that can be used in all other sections for  convenience. For instance, you can set _main_dir: ../testing_ so that _main_dir_ can be used in the rest of the sections.

Below is an example configuration file with one input relational source. In this case `DataSource1` is the only data source section, but other data sources can be considered by including additional sections. **[Here](https://github.com/morph-kgc/morph-kgc/blob/main/examples/configuration-file/default_config.ini)** you can find a configuration file which is more complete.

``` ini
[DEFAULT]
main_dir: ../testing

[CONFIGURATION]
output_file: knowledge-graph

[DataSource1]
mappings: ${main_dir}/mapping_file.rml.ttl
db_url: mysql+pymysql://user:password@localhost:3306/db_name
```

The parameters of the sections in the **[INI file](https://en.wikipedia.org/wiki/INI_file)** are explained below.

### Engine Configuration

The execution of Morph-KGC can be **tuned** via the **`CONFIGURATION`** section in the **[INI file](https://en.wikipedia.org/wiki/INI_file)**. This section can be empty, in which case Morph-KGC will use the **default** property values.

| <div style="width:195px">Property</div> | Description                                                                                                                                                                                                                                  | Values                                                                                                                                                                  |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **`output_file`**                       | File to write the resulting knowledge graph to. The file extension is derived from `output_format`, so it does not need to be given.                                                                                                          | **Default:** _knowledge-graph_                                                                                                                                          |
| **`output_dir`**                        | Directory to write the resulting knowledge graph to. If it is specified, `output_file` will be ignored and multiple output files will generated, one for each mapping partition.                                                             | **Default:**                                                                                                                                                            |
| **`output_format`**                     | RDF serialization to use for the resulting knowledge graph.                                                                                                                                                                                  | **Valid:** _[N-TRIPLES](https://www.w3.org/TR/n-triples/)_, _[N-QUADS](https://www.w3.org/TR/n-quads/)_, _[JELLY](https://w3id.org/jelly/)_<br>**Default:** _[N-TRIPLES](https://www.w3.org/TR/n-triples/)_ |
| **`na_values`**                         | Set of values to be interpreted as _NULL_ when retrieving data from the input sources. The set of values must be separated by commas.                                                                                                        | **Default:** ,_nan_                                                                                                                                                     |
| **`literal_escaping_chars`**            | Set of characters to be escaped in the generation of literals. The set of characters must be separated by commas. The backslash is always escaped.                                                                                           | **Default:** _"_,_\n_,_\r_                                                                                                                                              |
| **`safe_percent_encoding`**             | Set of ASCII characters that should not be percent encoded. All characters are encoded by default.                                                                                                                                           | **Example:** _:/_<br>**Default:**                                                                                                                                       |
| **`udfs`**                              | File with Python user-defined functions to be called from _[RML-FNML](https://w3id.org/rml/fnml/spec)_.                                                                                                                                       | **Default:**                                                                                                                                                            |
| **`state_dir`**                         | Directory the shared context of _[stateful functions](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation)_ is persisted to. If it is not provided, a temporary directory is created and removed for every run.      | **Default:**                                                                                                                                                            |
| **`mapping_partitioning`**              | [Mapping partitioning](https://content.iospress.com/download/semantic-web/sw223135?id=semantic-web%2Fsw223135) algorithm to use. Mapping partitioning can also be disabled.                                                                  | **Valid:** _PARTIAL-AGGREGATIONS_, _MAXIMAL_, _no_, _false_, _off_, _0_<br>**Default:** _PARTIAL-AGGREGATIONS_                                                          |
| **`infer_sql_datatypes`**               | Infer datatypes for relational databases. If a [datatypeable term map](https://www.w3.org/TR/r2rml/#dfn-datatypeable-term-map) has a _[rml:datatype](http://w3id.org/rml/datatype)_ property, then the datatype will not be inferred.         | **Valid:** _yes_, _no_, _true_, _false_, _on_, _off_, _1_, _0_<br>**Default:** _no_                                                                                     |
| **`number_of_processes`**               | The number of processes to use. If _1_, Morph-KGC will use sequential processing (minimizing memory consumption), otherwise parallel processing is used (minimizing execution time).                                                         | **Default:** _2 * number of CPUs in the system_                                                                                                                         |
| **`logging_level`**                     | Sets the [level](https://docs.python.org/3/library/logging.html#logging-levels) of the log messages to show.                                                                                                                                 | **Valid:** _DEBUG_, _INFO_, _WARNING_, _ERROR_, _CRITICAL_, _NOTSET_<br>**Default:** _INFO_                                                                             |
| **`logging_file`**                      | If not provided, log messages will be redirected to _stdout_. If a file path is provided, log messages will be written to the file.                                                                                                          | **Default:**                                                                                                                                                            |

{==

*__Note:__ there are some configuration properties that are ignored when using Morph-KGC as a [library](https://morph-kgc.readthedocs.io/en/latest/documentation/#library), such as `output_file`. Parallel processing is only available on Linux when running as a library, so `number_of_processes` is forced to _1_ on macOS and Windows; use the [command line](https://morph-kgc.readthedocs.io/en/latest/documentation/#command-line) for multi-process materialization there.*

==}

### Data Sources

One data source section should be included in the **[INI file](https://en.wikipedia.org/wiki/INI_file)** for each data source to be materialized. The properties in the data source section vary depending on the data source type (relational database or data file). **Remote** mapping files are supported.

{==

*__Note:__ Morph-KGC is case sensitive regarding identifiers. This means that table, column and reference names in the mappings must be the same as those in the data sources (no matter if the mapping uses [delimited identifiers](https://www.w3.org/TR/r2rml/#dfn-sql-identifier)).*

==}

#### Relational Databases

The properties to be specified for **relational databases** are listed below. All of the properties are **required**.

|<div style="width:100px">Property</div>|Description|<div style="width:475px">Values</div>|
|-------|-------|-------|
|**`mappings`**|Specifies the mapping file(s) or URL(s) for the relational database.|**[REQUIRED]**<br>**Valid:**<br>- The path to a mapping file or URL.<br>- The paths to multiple mapping files or URLs separated by commas.<br>- The path to a directory containing all the mapping files.|
|**`db_url`**|It is a URL that configures the database engine (username, password, hostname, database name). See **[here](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls)** how to create the database URLs. `{ENV_VAR}` placeholders are replaced with environment variables, so credentials need not be written to the file.|**[REQUIRED]**<br>**Example:** _dialect+driver://username:password@host:port/db_name_|
|**`connect_args`**|A dictionary string of options for [SQLAlchemy](https://www.sqlalchemy.org/). See [here](https://docs.sqlalchemy.org/en/20/core/engines.html#sqlalchemy.create_engine.params.connect_args) the [SQLAlchemy](https://www.sqlalchemy.org/) documentation.|**Example:** _{"http_path": "<cluster_http_path>"}_|

Example **`db_url`** values (see **[here](https://docs.sqlalchemy.org/en/20/core/engines.html#database-urls)** all the information):

- **[MySQL](https://www.mysql.com/)**: _mysql+pymysql://username:password@host:port/db_name_
- **[PostgreSQL](https://www.postgresql.org/)**: _postgresql+psycopg://username:password@host:port/db_name_
- **[Microsoft SQL Server](https://www.microsoft.com/sql-server)**: _mssql+pymssql://username:password@host:port/db_name_
- **[MariaDB](https://mariadb.org/)**: _mariadb+pymysql://username:password@host:port/db_name_
- **[Oracle](https://www.oracle.com/database/)**: _oracle+oracledb://username:password@host:port[/db_name][?service_name=<service>[&key=value&key=value...]]_
- **[SQLite](https://www.sqlite.org)**: _sqlite:///db_path/db_name.db_
- **[Databricks](https://www.databricks.com/)**: _databricks://token:<access_token>@<host>?http_path=<http_path>&catalog=<catalog>&schema=<schema>_
- **[Snowflake](https://www.snowflake.com/)**: _snowflake://<user_login_name>:<password>@<account_name>_
- **[Neo4j](https://neo4j.com/)**: _neo4j://host:port@username:password/db_name_

**[Neo4j](https://neo4j.com/)** is a **property graph database** rather than a relational one. It is configured in the very same way, with a `db_url`, and its logical sources use **[Cypher](https://neo4j.com/docs/cypher-manual/current/introduction/)** queries with the _rml:Cypher_ reference formulation.

#### Data Files

The properties to be specified for **data files** are listed below. **Remote** data files are supported. The `mappings` property is **required**.

|<div style="width:70px">Property</div>|Description|<div style="width:475px">Values</div>|
|-------|-------|-------|
|**`mappings`**|Specifies the mapping file(s) or URL(s) for the data file.|**[REQUIRED]**<br>**Valid:**<br>- The path to a mapping file or URL.<br>- The paths to multiple mapping files or URLs separated by commas.<br>- The path to a directory containing all the mapping files.|
|**`file_path`**|Specifies the local path or URL of the data file. It is **optional** since it can be provided within the mapping file with _[rml:source](http://w3id.org/rml/source)_. If it is provided it will **override** the local path or URL provided in the mapping files.|**Default:**|

{==

*__Note:__ [CSV](https://en.wikipedia.org/wiki/Comma-separated_values), [TSV](https://en.wikipedia.org/wiki/Tab-separated_values), [Stata](https://www.stata.com/) and [SAS](https://www.sas.com) support compressed files (gzip, bz2, zip, xz, tar). Files are decompressed _on-the-fly_ and compression format is automatically inferred.*

==}

### Resources

A **resource** is something the engine accesses while materializing, but that is not itself materialized: the **[SKOS](https://www.w3.org/TR/skos-reference/)** vocabulary a value is reconciled against, the **[SPARQL](https://www.w3.org/TR/sparql11-query/)** endpoint it is looked up in, the lookup table a **[stateful function](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation)** of your own reads.

Each one is declared in a **`[RESOURCE:<name>]`** section. The mapping only **names** the resource, while its location, credentials and access options stay in the configuration file. The same mapping therefore runs unchanged against a local copy of a vocabulary, a staging server or production.

``` ini
[RESOURCE:disease_vocabulary]
resource_type: SKOS_VOCABULARY
url: https://example.org/vocabulary/disease
username: {VOCABULARY_USER}
password: {VOCABULARY_PASSWORD}
matching: CASE-INSENSITIVE
```

The properties common to every resource type are listed below. `{ENV_VAR}` placeholders in `url`, `username` and `password` are replaced with **environment variables**, so credentials need not be written to the file at all.

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`resource_type`**|The kind of resource. The built-in [reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation) functions understand _SKOS_VOCABULARY_ and _SPARQL_ENDPOINT_; any other value is accepted for resource types declared by your own stateful functions.|**Example:** _SKOS_VOCABULARY_|
|**`url`**|Where the resource is downloaded from, or the endpoint queried. A local path is read from disk.|**Example:** _https://example.org/vocabulary/disease_|
|**`iri`**|The IRI identifying the resource, when it differs from `url`. A mapping may name the resource by it.|**Default:**|
|**`username`**, **`password`**|[HTTP Basic Authentication](https://datatracker.ietf.org/doc/html/rfc7617) credentials.|**Default:**|
|**`matching`**|How a value is matched against the indexed ones. _CASE-INSENSITIVE_ also collapses whitespace.|**Valid:** _EXACT_, _CASE-INSENSITIVE_<br>**Default:** _EXACT_|
|**`attributes`**|Comma-separated attributes to index when the mapping names none.|**Default:** the [SKOS](https://www.w3.org/TR/skos-reference/) labelling properties|
|**`timeout`**|Seconds to wait for the resource.|**Default:** _30_|

Only for **`SKOS_VOCABULARY`**:

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`format`**|RDF serialization of the vocabulary. It is guessed from the response and the URL when omitted.|**Valid:** _turtle_, _xml_, _nt_, _nquads_, _trig_, _json-ld_, _n3_|

Only for **`SPARQL_ENDPOINT`**:

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`query`**|The SELECT query the index is built from. It defaults to a query over the [SKOS](https://www.w3.org/TR/skos-reference/) labelling properties.|**Default:**|
|**`method`**|HTTP method used to query the endpoint.|**Valid:** _GET_, _POST_<br>**Default:** _GET_|
|**`concept_variable`**, **`attribute_variable`**, **`value_variable`**|Names of the variables projected by the query.|**Default:** _concept_, _attribute_, _value_|

## Advanced Setup

### Relational Databases

The supported DBMSs are **[MySQL](https://www.mysql.com/)**, **[PostgreSQL](https://www.postgresql.org/)**, **[Oracle](https://www.oracle.com/database/)**, **[Microsoft SQL Server](https://www.microsoft.com/sql-server)**, **[MariaDB](https://mariadb.org/)** and **[SQLite](https://www.sqlite.org)**. To use relational databases it is necessary to additionally install **DBAPI drivers**. You can install them via:

- **[MySQL](https://www.mysql.com/)** and **[MariaDB](https://mariadb.org/)**: `pip install morph-kgc[mysql]`.
- **[PostgreSQL](https://www.postgresql.org/)**: `pip install morph-kgc[postgresql]`.
- **[Microsoft SQL Server](https://www.microsoft.com/sql-server)**: `pip install morph-kgc[mssql]`.
- **[Oracle](https://www.oracle.com/database/)**: `pip install morph-kgc[oracle]`.
- **[SQLite](https://www.sqlite.org)**: `pip install morph-kgc[sqlite]`.
- **[Neo4j](https://neo4j.com)**: `pip install morph-kgc[neo4j]`.
- **[Snowflake](https://www.snowflake.com/)**: `pip install morph-kgc[snowflake]`.
- **[Databricks](https://www.databricks.com/)**: `pip install morph-kgc[databricks]`.

### Tabular Files

The supported tabular files formats are **[CSV](https://en.wikipedia.org/wiki/Comma-separated_values)**, **[TSV](https://en.wikipedia.org/wiki/Tab-separated_values)**, **[Excel](https://www.microsoft.com/en-us/microsoft-365/excel)**, **[Parquet](https://parquet.apache.org/documentation/latest/)**, **[Feather](https://arrow.apache.org/docs/python/feather.html)**, **[ORC](https://orc.apache.org/)**, **[Stata](https://www.stata.com/)**, **[SAS](https://www.sas.com)**, **[SPSS](https://www.ibm.com/analytics/spss-statistics-software)** and **[ODS](https://en.wikipedia.org/wiki/OpenDocument)**. To work with some of them it is necessary to install additional dependencies. You can install them via:

- **[Excel](https://www.microsoft.com/en-us/microsoft-365/excel)** and **[ODS](https://en.wikipedia.org/wiki/OpenDocument)**: `pip install morph-kgc[excel]`.
- **[Parquet](https://parquet.apache.org/documentation/latest/)**, **[Feather](https://arrow.apache.org/docs/python/feather.html)** and **[ORC](https://orc.apache.org/)**: `pip install morph-kgc[tabular]`.
- **[SPSS](https://www.ibm.com/analytics/spss-statistics-software)**: `pip install morph-kgc[spss]`.

### Geospatial Files

**[GeoParquet](https://geoparquet.org/)** and **[Shapefiles](https://en.wikipedia.org/wiki/Shapefile)** are read with **[GeoPandas](https://geopandas.org)**, which is installed via `pip install morph-kgc[geoparquet]`. Both are selected with the reference formulation of the logical source, _rml:GeoParquet_ and _rml:Shapefile_ respectively, and their geometries are handed to the mapping as **[WKT](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)** strings in a `geometry` reference.

``` turtle
<#TM1>
    rml:logicalSource [
        rml:source "data.parquet";
        rml:referenceFormulation rml:GeoParquet;
    ];
    rml:subjectMap [
        rml:template "http://example.com/{id}";
    ];
    rml:predicateObjectMap [
        rml:predicate geo:asWKT;
        rml:objectMap [
            rml:reference "geometry";
            rml:datatype geo:wktLiteral;
        ];
    ].
```

### Hierarchical Files

The supported hierarchical files formats are **[XML](https://www.w3.org/TR/xml/)** and **[JSON](https://www.json.org)**.

Morph-KGC uses **[XPath 3.0](https://www.w3.org/TR/xpath30/)** to query XML files and **[JSONPath](https://goessner.net/articles/JsonPath/)** to query JSON files. The specific JSONPath syntax supported by Morph-KGC can be consulted __[here](https://github.com/zhangxianbing/jsonpath-python#jsonpath-syntax)__.

### Output Formats

**[N-Triples](https://www.w3.org/TR/n-triples/)** and **[N-Quads](https://www.w3.org/TR/n-quads/)** need no additional dependency. **[Jelly](https://w3id.org/jelly/)**, a binary RDF serialization designed for large graphs and streaming, is written with **[pyjelly](https://w3id.org/jelly/pyjelly)** and is installed via `pip install morph-kgc[jelly]`.

{==

*__Note:__ [Jelly](https://w3id.org/jelly/) is serialized in one shot rather than streamed partition by partition, so materializing to it holds the whole knowledge graph in memory.*

==}

## Docker

You can also use Morph-KGC with the provided [Dockerfile](https://github.com/morph-kgc/morph-kgc/blob/main/Dockerfile).

### Image Building

Build the container as follows:

``` bash
docker build -t morph-kgc .
```

To include optional dependencies, use the `optional_dependencies` option as follows:

``` bash
docker build -t morph-kgc --build-arg optional_dependencies="sqlite,jelly" .
```

### Execution

The container is designed to mount a local directory containing the required files. To run the container, use the following command, replacing `$(pwd)/files` with the path to the local directory containing your files:

```bash
docker run -v $(pwd)/files:/app/files morph-kgc files/config.ini
```

This will mount the local directory to `/app/files` within the container and execute the application using the provided configuration file.

## Bootstrapping

Morph-KGC can **bootstrap** a mapping from the schema of a relational database, following the **[Direct Mapping](https://www.w3.org/TR/rdb-direct-mapping/)** of the database to RDF. It inspects the tables, primary keys and foreign keys, and writes a **[YARRRML](https://rml.io/yarrrml/spec/)** mapping you can then edit to target your own ontology.

``` bash
python -m morph_kgc.bootstrapping path/to/config.ini
```

The configuration file reuses the `db_url` of the data source section, which must be named `DataSource1`, and adds a `BOOTSTRAPPING` section:

``` ini
[CONFIGURATION]
output_dir: bootstrapped

[BOOTSTRAPPING]
base_iri: http://example.org/
build_mappings: yes

[DataSource1]
db_url: sqlite:///path/to/db_name.db
```

|<div style="width:120px">Property</div>|Description|<div style="width:300px">Values</div>|
|-------|-------|-------|
|**`base_iri`**|Base IRI of the generated terms.|**Default:** _http://example.org/_|
|**`build_mappings`**|Also materialize the knowledge graph with the generated mapping, instead of only writing it.|**Valid:** _yes_, _no_, _true_, _false_, _on_, _off_, _1_, _0_<br>**Default:** _no_|

The mapping is written to `direct_mapping.yaml` in `output_dir`.

![OEG](assets/logo-oeg.png){ width="150" align=left } ![UPM](assets/logo-upm.png){ width="161" align=right }

# Documentation

## Tutorial

You can get quickly started with the tutorial in **[`Google Colaboratory`](https://colab.research.google.com/drive/1ByFx_NOEfTZeaJ1Wtw3UwTH3H3-Sye2O?usp=sharing)**.

## Installation

In the following we describe different ways in which you can install and use Morph-KGC. Morph-KGC requires **[Python](https://www.python.org/) 3.10** or later.

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

Running the package as a module is equivalent:

``` bash
python -m morph_kgc path/to/config.ini
```

The configuration file must exist. **`-v`** (`--version`) prints the version of the engine and **`-h`** (`--help`) its usage:

``` bash
$ morph-kgc -v
Morph-KGC 2.10.0 | Copyright © 2020 Julián Arenas-Guerrero
```

**[In-memory data](https://morph-kgc.readthedocs.io/en/latest/rml/#in-memory-data)** can only be read when Morph-KGC is used as a [library](https://morph-kgc.readthedocs.io/en/latest/documentation/#library).

### Library

Morph-KGC can be used as a **library**, providing different methods to materialize the knowledge graph. It integrates with **[RDFLib](https://rdflib.readthedocs.io/en/stable/)** and **[Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)** to easily create and work with knowledge graphs in **[Python](https://www.python.org/)**.

The methods in the **API** accept the configuration as the **path to an INI file** (a string or a [`pathlib.Path`](https://docs.python.org/3/library/pathlib.html)), as an **INI string**, or as a **Python dictionary**.

``` python
import morph_kgc
from pathlib import Path

# the path to an INI file
config = '/path/to/config.ini'
# or
config = Path('/path/to/config.ini')

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

{==

*__Note:__ a string is read as a path only when that file exists, otherwise it is parsed as INI text: a mistyped path therefore fails with `configparser.MissingSectionHeaderError`. A `pathlib.Path` to a missing file loads no data source at all, and the run then fails with `ValueError: max() iterable argument is empty`. The values of a dictionary are converted to strings, so several `mappings` are given as one comma-separated string.*

==}

Every method takes an optional **second argument**, `python_source`: a dictionary with the **in-memory data structures** the mapping reads ([DataFrames](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html), dictionaries, lists, tuples and [JSON](https://www.json.org) strings), keyed by the name the logical sources give them. See **[In-Memory Data](https://morph-kgc.readthedocs.io/en/latest/rml/#in-memory-data)**.

``` python
import pandas as pd

films_df = pd.DataFrame({'id': [1, 2], 'title': ['Metropolis', 'M']})

graph = morph_kgc.materialize(config, {'films': films_df})
```

#### [RDFLib](https://rdflib.readthedocs.io/en/stable/)

**`morph_kgc.materialize(config)`**

Materialize the knowledge graph to **[RDFLib](https://rdflib.readthedocs.io/en/stable/)**. It returns an **`rdflib.Graph`**, or an **`rdflib.Dataset`** when `output_format` is _N-QUADS_, so that the statements in named graphs are kept.

``` python
# generate the triples and load them to an RDFLib graph

graph = morph_kgc.materialize(config)
# or
graph = morph_kgc.materialize('/path/to/config.ini')

# work with the RDFLib graph
q_res = graph.query(' SELECT DISTINCT ?classes WHERE { ?s a ?classes } ')
```

{==

*__Note:__ [RDFLib](https://rdflib.readthedocs.io/en/stable/) does not read [RDF 1.2](https://www.w3.org/TR/rdf12-concepts/) triple terms or directional language-tagged strings, hence `materialize` does not support the [RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification) constructs that generate them. Use `materialize_oxigraph` or `materialize_set`, or write the knowledge graph to an _N-TRIPLES_ or _N-QUADS_ file instead (_JELLY_ output goes through RDFLib and fails as well).*

==}

#### [Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)

**`morph_kgc.materialize_oxigraph(config)`**

Materialize the knowledge graph to **[Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)**. Oxigraph reads [RDF 1.2](https://www.w3.org/TR/rdf12-concepts/), so this method supports every [RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification) construct.

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

Materialize the knowledge graph to a Python **Set of triples**. Each element is a serialized statement without its trailing ` .` (with _N-QUADS_, a statement in the default graph keeps a trailing space).

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
| **`output_file`** | File to write the resulting knowledge graph to. The file extension is derived from `output_format`, so it does not need to be given: another extension is replaced, e.g. _out/kg.ttl_ is written as _out/kg.nt_. An existing file is overwritten. | **Default:** _knowledge-graph_ |
| **`output_dir`** | Directory to write the resulting knowledge graph to. If it is specified, `output_file` will be ignored (except for _JELLY_) and multiple output files will be generated, one for each mapping partition, named after it, e.g. _1-1-2-1.nq_. | **Default:** |
| **`output_format`** | RDF serialization to use for the resulting knowledge graph. [Graph maps](https://morph-kgc.readthedocs.io/en/latest/rml/#named-graphs) are only used with _N-QUADS_: with _N-TRIPLES_ and _JELLY_ every statement goes to the default graph. | **Valid:** _[N-TRIPLES](https://www.w3.org/TR/n-triples/)_, _[N-QUADS](https://www.w3.org/TR/n-quads/)_, _[JELLY](https://w3id.org/jelly/)_<br>**Default:** _[N-TRIPLES](https://www.w3.org/TR/n-triples/)_ |
| **`na_values`** | Set of values to be interpreted as _NULL_ when retrieving data from the input sources. The set of values must be separated by commas. A record with _NULL_ in a reference of a mapping rule generates no triple for that rule. With the default, empty strings are _NULL_ too: to keep them, list only other values, e.g. _nan_ (an empty setting still makes them _NULL_). | **Default:** ,_nan_ |
| **`literal_escaping_chars`**            | Additional set of characters to be escaped in the generation of literals. The set of characters must be separated by commas. The backslash, the double quote, the line feed and the carriage return are always escaped.                                                                                           | **Default:** _"_,_\n_,_\r_                                                                                                                                              |
| **`safe_percent_encoding`** | Set of ASCII characters that should not be percent encoded in the IRIs generated by templates. By default, every character other than the unreserved ones (letters, digits, `-`, `.`, `_` and `~`) is encoded. | **Example:** _:/_<br>**Default:** |
| **`udfs`**                              | File with Python user-defined functions to be called from _[RML-FNML](https://w3id.org/rml/fnml/spec)_.                                                                                                                                       | **Default:**                                                                                                                                                            |
| **`state_dir`** | Directory with room on disk for the shared contexts of _[stateful functions](https://morph-kgc.readthedocs.io/en/latest/rml/#stateful-functions-of-your-own)_, such as the indexes of the [reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation) functions. Every run creates a directory of its own inside it, `morph-kgc-state-*`, and removes it when the run ends, also when it fails, so nothing is kept across runs. It is created if missing, and anything else in it is left untouched. If it is not provided, the temporary directory of the system is used. | **Default:** |
| **`state_memory_limit`** | Memory **[DuckDB](https://duckdb.org/)** may use in **each process** to build and read the on-disk tables of the shared contexts of stateful functions, such as the reconciliation indexes. Past it, DuckDB spills to the directory of the run in `state_dir`. Total memory therefore grows with `number_of_processes`, not with the size of the contexts. | **Example:** _2GB_<br>**Default:** _512MB_ |
| **`reconciliation_report`** | [JSON Lines](https://jsonlines.org) file the values the [reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation) functions match to no entity, or to several, are written to when the run ends, see **[Reconciliation Report](https://morph-kgc.readthedocs.io/en/latest/documentation/#reconciliation-report)**. If it is not provided, no report is written. | **Example:** _reports/reconciliation.jsonl_<br>**Default:** |
| **`mapping_partitioning`**              | [Mapping partitioning](https://content.iospress.com/download/semantic-web/sw223135?id=semantic-web%2Fsw223135) algorithm to use. Mapping partitioning can also be disabled.                                                                  | **Valid:** _PARTIAL-AGGREGATIONS_, _MAXIMAL_, _no_, _false_, _off_, _0_<br>**Default:** _PARTIAL-AGGREGATIONS_                                                          |
| **`infer_sql_datatypes`** | Infer the datatypes of literals from the SQL types of the columns of relational databases, e.g. _xsd:integer_ for an _INT_ column (string types add no datatype). It only applies to object maps with an _[rml:reference](http://w3id.org/rml/reference)_ over an _[rml:tableName](http://w3id.org/rml/tableName)_ logical source, not to _[rml:query](http://w3id.org/rml/query)_ sources. If a [datatypeable term map](https://www.w3.org/TR/r2rml/#dfn-datatypeable-term-map) has a _[rml:datatype](http://w3id.org/rml/datatype)_ or a language tag, then the datatype will not be inferred. | **Valid:** _yes_, _no_, _true_, _false_, _on_, _off_, _1_, _0_<br>**Default:** _no_ |
| **`number_of_processes`** | The number of processes to use. If _1_, Morph-KGC will use sequential processing (minimizing memory consumption), otherwise parallel processing is used (minimizing execution time). Each process holds its own data, so if a worker is killed for lack of memory the run fails and asks to lower it. | **Valid:** _1_ or more<br>**Default:** _2 * number of CPUs in the system_ |
| **`api_token`** | Python module with a `get_api_token(token_name)` function, handing out the tokens the headers of **[HTTP API](https://morph-kgc.readthedocs.io/en/latest/documentation/#http-apis)** sources name. A relative path is relative to the current working directory. | **Example:** _api_token.py_<br>**Default:** |
| **`logging_level`**                     | Sets the [level](https://docs.python.org/3/library/logging.html#logging-levels) of the log messages to show.                                                                                                                                 | **Valid:** _DEBUG_, _INFO_, _WARNING_, _ERROR_, _CRITICAL_, _NOTSET_<br>**Default:** _INFO_                                                                             |
| **`logging_file`**                      | If not provided, log messages will be redirected to _stderr_. If a file path is provided, log messages will be written to the file.                                                                                                          | **Default:**                                                                                                                                                            |

{==

*__Note:__ there are some configuration properties that are ignored when using Morph-KGC as a [library](https://morph-kgc.readthedocs.io/en/latest/documentation/#library), such as `output_file`. Parallel processing is only available on **Linux**: on macOS and Windows mapping rules are always materialized sequentially, from the [command line](https://morph-kgc.readthedocs.io/en/latest/documentation/#command-line) as well as from the library.*

==}

### Data Sources

One data source section should be included in the **[INI file](https://en.wikipedia.org/wiki/INI_file)** for each data source to be materialized. The properties in the data source section vary depending on the data source type (relational database, data file or HTTP API). **Remote** mapping files are supported, except [YARRRML](https://morph-kgc.readthedocs.io/en/latest/rml/#yarrrml) ones, which must be local files.

{==

*__Note:__ Morph-KGC is case sensitive regarding identifiers. This means that table, column and reference names in the mappings must be the same as those in the data sources (no matter if the mapping uses [delimited identifiers](https://www.w3.org/TR/r2rml/#dfn-sql-identifier)). The generated SQL quotes identifiers the way the database expects: backticks for MySQL, MariaDB and Databricks, brackets for SQL Server and double quotes for the others. A dotted _[rml:tableName](http://w3id.org/rml/tableName)_ such as _myschema.Student_ is read as a table of a schema, and backtick-quoted identifiers in an _[rml:query](http://w3id.org/rml/query)_ are rewritten to the quoting of the database, except for Databricks, where they are removed.*

==}

#### Relational Databases

The properties to be specified for **relational databases** are listed below. `mappings` and `db_url` are **required**.

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

**[Neo4j](https://neo4j.com/)** is a **property graph database** rather than a relational one. It is configured in the very same way, with a `db_url` (`connect_args` does not apply), and its logical sources use **[Cypher](https://neo4j.com/docs/cypher-manual/current/introduction/)** queries with the _rml:Cypher_ reference formulation (a GQL reference formulation is accepted as well). In a section with a `db_url`, a logical source with any other reference formulation, or none, is read as SQL.

#### Data Files

The properties to be specified for **data files** are listed below. **Remote** data files are supported. The `mappings` property is **required**.

|<div style="width:70px">Property</div>|Description|<div style="width:475px">Values</div>|
|-------|-------|-------|
|**`mappings`**|Specifies the mapping file(s) or URL(s) for the data file.|**[REQUIRED]**<br>**Valid:**<br>- The path to a mapping file or URL.<br>- The paths to multiple mapping files or URLs separated by commas.<br>- The path to a directory containing all the mapping files.|
|**`file_path`**|Specifies the local path or URL of the data file. It is **optional** since it can be provided within the mapping file with _[rml:source](http://w3id.org/rml/source)_. If it is provided it will **override** the local path or URL provided in the mapping files.|**Default:**|

{==

*__Note:__ [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) and [TSV](https://en.wikipedia.org/wiki/Tab-separated_values) files can be compressed (gzip, bz2, zip, xz, tar). Files are decompressed _on-the-fly_ and compression format is automatically inferred. Since the type of a file is otherwise taken from its extension, the logical source of a compressed file must declare its reference formulation, _rml:CSV_ or _rml:TSV_.*

==}

#### HTTP APIs

A logical source can also read the **[JSON](https://www.json.org)** an **HTTP API** answers with. The request is described in the mapping with the **[HTTP Vocabulary in RDF](https://www.w3.org/TR/HTTP-in-RDF10/)**, see **[Web APIs](https://morph-kgc.readthedocs.io/en/latest/rml/#web-apis)**, so the data source section only needs `mappings`. The `api_token` property of the **[engine configuration](https://morph-kgc.readthedocs.io/en/latest/documentation/#engine-configuration)** is only needed for tokens that have to be asked for:

``` ini
[CONFIGURATION]
api_token: api_token.py

[DataSource1]
mappings: mapping.ttl
```

Only the `Authorization`, `Accept`, `KeyId` and `User-Agent` fields of the request are sent as headers; every other field is sent as a query parameter. So that tokens need not be written to the mapping, the value of each field is resolved before the request is sent:

1. The **name of an environment variable** is replaced with its value.
2. A **`{ENV_VAR}` template**, such as _Bearer {FILMS_TOKEN}_, is filled with environment variables. If one of them is not set, the run fails and no request is sent.
3. Otherwise, if `api_token` is set, the value is passed to the **`get_api_token`** function of that module, and the token it returns, if any, is sent.
4. Any other value is sent as it is written, e.g. _application/json_ for `Accept`.

``` python
def get_api_token(token_name):
    # called with the htv:fieldValue of every header of the request
    if token_name == 'films_api':
        with open('films_api.token') as token_file:
            return 'Bearer ' + token_file.read().strip()
    return None  # any other value is sent as it is written
```

The module is run again for every value it resolves, so a token that is costly to obtain should be cached outside it, e.g. in a file, as the **[example](https://github.com/morph-kgc/morph-kgc/blob/main/examples/http-api/README.md)** does. An answer that is not JSON, or an HTTP error status, fails the run.

{==

*__Note:__ the API is requested once for every mapping rule that reads it, roughly one per predicate-object map of the triples map plus one per join that reads it as a parent, rather than once per run: keep it in mind for an API that is rate limited or slow to answer, and make sure it gives the same answer every time.*

==}

### Resources

A **resource** is something the engine accesses while materializing, but that is not itself materialized: the **[SKOS](https://www.w3.org/TR/skos-reference/)** vocabulary a value is reconciled against, the **[SPARQL](https://www.w3.org/TR/sparql11-query/)** endpoint it is looked up in, the lookup table a **[stateful function](https://morph-kgc.readthedocs.io/en/latest/rml/#stateful-functions-of-your-own)** of your own reads.

Each one is declared in a **`[RESOURCE:<name>]`** section. The mapping only **names** the resource, while its location, credentials and access options stay in the configuration file. The same mapping therefore runs unchanged against a local copy of a vocabulary, a staging server or production. With the section below, the **[reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation)** example of the RML page reads its `genres` vocabulary from a server that asks for credentials, and the mapping does not change:

``` ini
[RESOURCE:genres]
resource_type: SKOS_VOCABULARY
url: https://example.com/vocabulary/genres
username: {VOCABULARY_USER}
password: {VOCABULARY_PASSWORD}
```

How a value is matched against the resource (`morph-fn:matching`), and what a value matching several of its entities yields (`morph-fn:ambiguity`), are set in the mapping, since they change the triples generated: see **[Reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation)**.

The properties common to every resource type are listed below. `{ENV_VAR}` placeholders in `url`, `iri`, `username` and `password` are replaced with **environment variables**, so credentials need not be written to the file at all. A variable that is not set is an error, and a literal `{` or `}` in these properties is written `{{` or `}}`. Credentials written inside a URL, as in _https://user:password@example.com_, are rejected. A misspelled property is ignored without a warning, and a resource name declared twice is an error.

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`resource_type`**|The kind of resource, case-insensitive. The built-in [reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation) functions understand _SKOS_VOCABULARY_ and _SPARQL_ENDPOINT_; any other value is accepted for resource types declared by your own [stateful functions](https://morph-kgc.readthedocs.io/en/latest/rml/#stateful-functions-of-your-own).|**[REQUIRED]**<br>**Example:** _SKOS_VOCABULARY_|
|**`url`**|Where the resource is read from. For a _SKOS_VOCABULARY_, a URL or a local path, relative to the current working directory (not to the configuration file); compressed files (_.gz_, _.zip_) are not decompressed, though a server's gzip encoding is. For a _SPARQL_ENDPOINT_, the HTTP(S) URL the endpoint is queried at.|**Example:** _https://example.com/vocabulary/genres_|
|**`iri`**|The IRI identifying the resource, when it differs from `url`. A mapping may name the resource by it. For a _SPARQL_ENDPOINT_, it is the endpoint URL the mapping names, so that a mapping naming a production endpoint runs against the one in `url`.|**Default:**|
|**`username`**, **`password`**|[HTTP Basic Authentication](https://datatracker.ietf.org/doc/html/rfc7617) credentials.|**Default:**|
|**`timeout`**|Seconds to wait for the resource.|**Default:** _30_ for a _SKOS_VOCABULARY_, _300_ for a _SPARQL_ENDPOINT_|
|**`ambiguity`**|What a value matching several entities of a _SKOS_VOCABULARY_ or a _SPARQL_ENDPOINT_ yields, when the function execution sets no `morph-fn:ambiguity`: one triple per entity (_ALL_), no triple (_NONE_), or an error aborting the run (_FAIL_).|**Valid:** _ALL_, _NONE_, _FAIL_<br>**Default:** _ALL_|

Only for **`SKOS_VOCABULARY`**:

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`format`**|RDF serialization of the vocabulary. It is guessed from the response and the URL when omitted. A vocabulary serialized as quads is indexed across all its named graphs.|**Valid:** _turtle_, _xml_, _nt_, _nquads_, _trig_, _json-ld_, _n3_, or a media type such as _text/turtle_|
|**`attributes`**|Comma-separated attributes a value is matched against when the function execution names none with `morph-fn:attributeIRI`. They are full IRIs, or prefixed names with the prefixes _skos_, _rdf_, _rdfs_, _owl_, _xsd_, _dc_, _dcterms_ (or _dct_), _schema_ (_https://schema.org/_) or _foaf_.|**Example:** _skos:prefLabel,skos:altLabel_<br>**Default:** every property whose value is a literal|

Only for **`SPARQL_ENDPOINT`**:

|<div style="width:110px">Property</div>|Description|<div style="width:400px">Values</div>|
|-------|-------|-------|
|**`method`**|HTTP method used to query the endpoint.|**Valid:** _GET_, _POST_<br>**Default:** _GET_, or _POST_ for a query too long for a URL|

The endpoint and the query of a **[SPARQL reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#against-a-sparql-endpoint)** are written in the mapping, so a `SPARQL_ENDPOINT` section is **optional**: an endpoint open to everyone needs none. Declare one for the credentials of an endpoint, or for another `timeout`, `method` or `ambiguity`. The section applies only when its `url` or `iri` equals the endpoint URL of the mapping **character for character**, a trailing `/` included. Otherwise the endpoint is queried without it, so without its credentials, and only an _INFO_ log line says so. Two sections declaring the same endpoint are an error.

Below, the mapping names the production endpoint, _https://example.com/sparql_, declared as `iri`, while the endpoint queried is a local copy of it:

``` ini
[RESOURCE:genres_endpoint]
resource_type: SPARQL_ENDPOINT
iri: https://example.com/sparql
url: http://127.0.0.1:8890/sparql
username: {ENDPOINT_USER}
password: {ENDPOINT_PASSWORD}
method: POST
```

#### Reconciliation Report

The values the reconciliation functions match to no entity, or to several, are never logged. When `reconciliation_report` is set in the **[engine configuration](https://morph-kgc.readthedocs.io/en/latest/documentation/#engine-configuration)**, they are written to that file instead, in **[JSON Lines](https://jsonlines.org)** (one JSON object per line), so that the values left without a triple can be reviewed:

``` ini
[CONFIGURATION]
reconciliation_report: reports/reconciliation.jsonl
```

The report holds one line per distinct execution, value and outcome, whatever the number of records, mapping rules and processes involved. It is written when the run ends, also when it is aborted by the _FAIL_ ambiguity policy, and when Morph-KGC is used as a [library](https://morph-kgc.readthedocs.io/en/latest/documentation/#library). The file is overwritten on every run, and left empty when there is nothing to report. A relative path is relative to the current working directory. Its directory is created when the configuration is read, and a path that names a directory, or cannot be written to, is an error then. The fields of each line are:

| Field | Meaning |
|-------|-------|
|**`outcome`**|_NO_MATCH_ for a value matching no entity, _MULTIPLE_MATCH_ for one matching several.|
|**`value`**, **`normalized_value`**|The value as read, and as looked up after `morph-fn:matching` was applied. Arrays for an entity matched by several values.|
|**`function_iri`**, **`execution_id`**|The function and its execution in the mapping.|
|**`triples_map`**|The triples map reconciling the value.|
|**`resource`**|The resource reconciled against. For a SPARQL endpoint, **`endpoint`** and **`query_hash`** (the SHA-256 of the query) instead.|
|**`candidates`**|For _MULTIPLE_MATCH_, the entities the value matches.|
|**`source`**, **`mapping`**|The logical source of the triples map, and the mapping file declaring it.|

Whether or not there is a report, a summary of each execution is logged at _INFO_ level. For the [reconciliation](https://morph-kgc.readthedocs.io/en/latest/rml/#reconciliation) example of the RML page, whose `Drama` film matches no genre, the execution `handled 3 value(s): 2 matched, 1 unmatched (1 distinct), 0 ambiguous.`, and its report holds a single _NO_MATCH_ line, for the value `Drama`.

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

**[GeoParquet](https://geoparquet.org/)** and **[Shapefiles](https://en.wikipedia.org/wiki/Shapefile)** are read with **[GeoPandas](https://geopandas.org)**, which is installed via `pip install morph-kgc[geoparquet]`. A GeoParquet file is selected with the _rml:GeoParquet_ reference formulation of the logical source (without it, a _.parquet_ file is read as plain Parquet, with binary geometries), and a Shapefile by its _.shp_ extension. Their geometries are handed to the mapping as **[WKT](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)** strings in a `geometry` reference.

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

Morph-KGC uses **[XPath 3.0](https://www.w3.org/TR/xpath30/)** to query XML files and **[JSONPath](https://goessner.net/articles/JsonPath/)** to query JSON files. The specific JSONPath syntax supported by Morph-KGC can be consulted __[here](https://github.com/zhangxianbing/jsonpath-python#jsonpath-syntax)__. Their logical sources require an _[rml:iterator](http://w3id.org/rml/iterator)_. _GeoJSON_ files are read as JSON.

XPath 3.0 is used for the iterator of XML files, while the references of their term maps are simpler paths relative to the iterated element: child elements (_director/name_) and attributes (_@id_). The references of JSON files are relative to the iterated object, and may be written with or without a leading `$.` (_name_ or _$.name_), except for [HTTP APIs](https://morph-kgc.readthedocs.io/en/latest/documentation/#http-apis), whose references are written without it.

### Output Formats

**[N-Triples](https://www.w3.org/TR/n-triples/)** and **[N-Quads](https://www.w3.org/TR/n-quads/)** need no additional dependency. **[Jelly](https://w3id.org/jelly/)**, a binary RDF serialization designed for large graphs and streaming, is written with **[pyjelly](https://w3id.org/jelly/pyjelly)** and is installed via `pip install morph-kgc[jelly]`.

{==

*__Note:__ [Jelly](https://w3id.org/jelly/) is serialized in one shot rather than streamed partition by partition, so materializing to it holds the whole knowledge graph in memory. It is written by the [command line](https://morph-kgc.readthedocs.io/en/latest/documentation/#command-line) to a single file, `output_file` with the _.jelly_ extension (inside `output_dir`, if it is given). [Graph maps](https://morph-kgc.readthedocs.io/en/latest/rml/#named-graphs) are not used, as with N-Triples. Since the graph is built with [RDFLib](https://rdflib.readthedocs.io/en/stable/), the [RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification) triple terms and directional language-tagged strings cannot be written to Jelly. The [library](https://morph-kgc.readthedocs.io/en/latest/documentation/#library) methods return a graph or a set of triples instead.*

==}

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

The mapping is written to `direct_mapping.yaml` in `output_dir`, or in the current working directory when `output_dir` is not given. With `build_mappings`, the knowledge graph is written next to it, to `output_file` (_output_graph_ by default) in the `output_format` of the `CONFIGURATION` section.

Bootstrapping needs the **[SQLAlchemy](https://www.sqlalchemy.org/)** driver of the database (see **[Relational Databases](https://morph-kgc.readthedocs.io/en/latest/documentation/#relational-databases_1)**) and **[PyYAML](https://pyyaml.org/)**, which is not installed with Morph-KGC:

``` bash
pip install pyyaml
```

{==

*__Note:__ the configuration file of bootstrapping is read without `${...}` variables, and `{ENV_VAR}` placeholders are not replaced in its `db_url`. A `%` in it, as in a URL-encoded password, must be written `%%`.*

==}

![OEG](assets/logo-oeg.png){ width="150" align=left } ![UPM](assets/logo-upm.png){ width="161" align=right }

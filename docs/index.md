<figure markdown>
  ![Morph-KGC](assets/logo.png){ width="300" }
</figure>

#

**Morph-KGC** is an engine that constructs **[RDF](https://www.w3.org/TR/rdf11-concepts/)** knowledge graphs from heterogeneous data sources with the **[R2RML](https://www.w3.org/TR/r2rml/)** and **[RML](https://w3id.org/rml/core/spec)** mapping languages. Morph-KGC is built on top of [pandas](https://pandas.pydata.org/) and it leverages *mapping partitions* to significantly reduce execution times and memory consumption for large data sources.

## Features

- User-friendly mappings with **[YARRRML](https://rml.io/yarrrml/spec/)**.
- Transformation functions with **[RML-FNML](https://w3id.org/rml/fnml/spec)**, including **Python UDFs**.
- [RDF-star](https://w3c.github.io/rdf-star/cg-spec/2021-12-17.html) generation with **[RML-star](https://w3id.org/rml/star/spec)**.
- **[RML views](https://2023.eswc-conferences.org/wp-content/uploads/2023/05/paper_Arenas-Guerrero_2023_Boosting.pdf)** over tabular data sources and [JSON](https://www.json.org) files.
- Integration with **[RDFLib](https://rdflib.readthedocs.io)**, **[Oxigraph](https://pyoxigraph.readthedocs.io/en/latest/)** and [Kafka](https://kafka-python.readthedocs.io).
- **Optimized** to materialize large knowledge graphs.
- **Remote** data and mapping files.
- Input data formats:
    - **Relational databases**: [MySQL](https://www.mysql.com/), [PostgreSQL](https://www.postgresql.org/), [Oracle](https://www.oracle.com/database/), [Microsoft SQL Server](https://www.microsoft.com/sql-server), [MariaDB](https://mariadb.org/), [SQLite](https://www.sqlite.org).
    - **Tabular files**: [CSV](https://en.wikipedia.org/wiki/Comma-separated_values), [TSV](https://en.wikipedia.org/wiki/Tab-separated_values), [Excel](https://www.microsoft.com/en-us/microsoft-365/excel), [Parquet](https://parquet.apache.org/documentation/latest/), [Feather](https://arrow.apache.org/docs/python/feather.html), [ORC](https://orc.apache.org/), [Stata](https://www.stata.com/), [SAS](https://www.sas.com), [SPSS](https://www.ibm.com/analytics/spss-statistics-software), [ODS](https://en.wikipedia.org/wiki/OpenDocument).
    - **Hierarchical files**: [JSON](https://www.json.org), [XML](https://www.w3.org/TR/xml/).
    - **In-memory data structures**: [Python Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries), [DataFrames](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html).
    - **Cloud data lake solutions**: [Databricks](https://www.databricks.com/), [Snowflake](https://www.snowflake.com/).
    - **Property graph databases**: [Neo4j](https://neo4j.com/), [Kùzu](https://kuzudb.com).

## Organizations that Use Morph-KGC

<figure markdown>
  ![BASF](assets/BASF.png){ width="300" align=left }
</figure>

## Licenses

**Morph-KGC** is available under the **[Apache License 2.0](https://github.com/morph-kgc/morph-kgc/blob/main/LICENSE)**.

The **documentation** is licensed under **[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)**.

## Author

- **[Julián Arenas-Guerrero](https://github.com/arenas-guerrero-julian/) - [julian.arenas.guerrero@upm.es](mailto:julian.arenas.guerrero@upm.es)**

*[Universidad Politécnica de Madrid](https://www.upm.es/internacional)*.

## Citing

If you used Morph-KGC in your work, please cite the **[SoftwareX](https://www.sciencedirect.com/science/article/pii/S2352711024000803)** or **[SWJ](https://www.doi.org/10.3233/SW-223135)** papers:

``` bib
@article{arenas2024morph,
  title     = {{Morph-KGC: Scalable knowledge graph materialization with mapping partitions}},
  author    = {Arenas-Guerrero, Julián and Chaves-Fraga, David and Toledo, Jhon and Pérez, María S. and Corcho, Oscar},
  journal   = {Semantic Web},
  year      = {2024},
  volume    = {15},
  number    = {1},
  pages     = {1-20},
  issn      = {2210-4968},
  publisher = {IOS Press},
  doi       = {10.3233/SW-223135}
}
```

![OEG](assets/logo-oeg.png){ width="150" align=left } ![UPM](assets/logo-upm.png){ width="161" align=right }

# FAQ

#### Who develops Morph-KGC?
Morph-KGC was created by **[Julián Arenas-Guerrero](https://github.com/ArenasGuerreroJulian/)** in the **[Ontology Engineering Group](https://oeg.fi.upm.es)** at **[Universidad Politécnica de Madrid](https://www.upm.es/internacional)**.

#### How can I learn R2RML, RML, RML-FNML or YARRRML?
The best starting point is the **[R2RML Recommendation](https://www.w3.org/TR/r2rml/)**. After that, you can continue with the **[RML-Core](https://w3id.org/rml/core/spec)**, **[RML-FNML](https://w3id.org/rml/fnml/spec)** or **[YARRRML](https://rml.io/yarrrml/spec/)** specifications.

#### I have some question related to R2RML, RML, RML-FNML or YARRRML
The best place to get the answer is at **[Discussions for RML questions](https://github.com/kg-construct/rml-questions/discussions)**.

#### What happened to RML-star?
The quoted triples of **RML-star** have been superseded by the triple terms and reifying triples of **[RDF 1.2](https://www.w3.org/TR/rdf12-concepts/)**. Morph-KGC now generates them with the **[RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification)** constructs _rml:tripleTermMap_, _rml:NonAssertedTriplesMap_ and _rml:reifyingMap_.

#### Why does `materialize` fail on a mapping with triple terms?
**[RDFLib](https://rdflib.readthedocs.io/en/stable/)** 7.x reads neither triple terms nor directional language-tagged strings. Materialize such mappings with `materialize_set`, or write the knowledge graph to a file. See the **[RML 1.2](https://morph-kgc.readthedocs.io/en/latest/rml/#triple-terms-and-reification)** section.

#### I installed Morph-KGC but I get an error saying some dependency is missing
To use relational databases, some data files and the [Jelly](https://w3id.org/jelly/) output format it is required to install additional dependencies. You can check specific configuration options in **[Advanced Setup](https://morph-kgc.readthedocs.io/en/latest/documentation/#advanced-setup)**.

#### Can I keep my credentials out of the configuration file?
Yes. `{ENV_VAR}` placeholders in `db_url`, and in the `url`, `username` and `password` of a **[resource](https://morph-kgc.readthedocs.io/en/latest/documentation/#resources)**, are replaced with the corresponding environment variables.

#### How do I submit a bug report?
Please, [open an issue](https://github.com/morph-kgc/morph-kgc/issues/new/choose) with a clear description of the bug. We will try to solve it as soon as possible.

#### I have a suggestion to improve the documentation
Please, [open an issue](https://github.com/morph-kgc/morph-kgc/issues/new/choose) providing a description of what documentation you believe needs to be fixed/improved and your suggestion. You can also contribute with a pull request to the **[Docs GitHub repository](https://github.com/morph-kgc/morph-kgc-docs)**.

#### Can I use Morph-KGC in a commercial project?
Yes! Morph-KGC is distributed under the **[Apache License 2.0](https://github.com/morph-kgc/morph-kgc/blob/main/LICENSE)** which allows  commercial use, modification, distribution, patent use and private use. Of course, we will appreciate it if you tell us that you are using Morph-KGC and can give us some details on how it is being used (as much as you can given confidentiality constraints). This would allow creating a database of projects where Morph-KGC is being used that could help us to request additional funding in the future, if needed.

#### I need commercial support
You are welcome to **[contact us](mailto:julian.arenas.guerrero@upm.es)**!

![OEG](assets/logo-oeg.png){ width="150" align=left } ![UPM](assets/logo-upm.png){ width="161" align=right }

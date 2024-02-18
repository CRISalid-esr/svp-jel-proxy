# SoVisu+ JEL proxy

This project is a variation over
the [Apache Jena Fuseki Docker Tools](https://jena.apache.org/documentation/fuseki2/fuseki-docker.html).
This project is a utility that allows injecting the [JEL](https://www.aeaweb.org/econlit/jelCodes.php) vocabulary
vocabulary at the time of building the Fuseki Docker image to obtain a turnkey Docker container that can be used as a
proxy to resolve JEL URLs.
The original source is not directly usable due to performance or availability limitations.
The project is primarily intended for use within the SoVisu+ project but can be used in any context where the JEL vocabulary is needed.

## Provided vocabulary

The [JEL](https://www.aeaweb.org/econlit/jelCodes.php) vocabulary is a widely used vocabulary to classify economic
literature.
Hal, the french open
archive, [uses it to classify documents related to the field of economics](https://doc.archives-ouvertes.fr/ufaqs/est-il-possible-dajouter-la-classification-jel/)
and it is also used by the [RePEc](https://ideas.repec.org/j/) project.

Hence the need for institutional research information management systems (RIMS) to use this vocabulary when trying
to comply with
semantic web standards. However, they encounter the issue of not having a platform that delivers the vocabulary in a
compliant format.

Indeed, the only available RDF version of JEL is published by Leibniz-Informationszentrum
Wirtschaft (https://zbw.eu/beta/skosmos/jel/en/) but the server capacities
are insufficient to handle on-the-fly requests from third-party applications.
As of now, the vocabulary download is unavailable on zbw.eu. However, the svp-voc-proxy was loaded from
the [Internet Archive Wayback Machine](https://web.archive.org/web/20240000000000*/https://zbw.eu/beta/external_identifiers/jel/download/jel.rdf.zip)
where
a [recent version of the vocabulary](https://web.archive.org/web/20240000000000*/https://zbw.eu/beta/external_identifiers/jel/download/jel.rdf.zip)
was accessible.

## Usage

### Build docker package

```bash
docker image build --build-arg JENA_VERSION=5.0.0-rc1 -t svp-jel-proxy:v0 .
```

### Run the docker container

```bash
docker container run --rm -it -p 3030:3030 --name svp-jel-proxy svp-jel-proxy:v0
```

### Use docker-compose

```bash
docker-compose up
```

### Test queries

We provide a few examples of queries that can be used to test the service using the `curl` command but you may feel more
comfortable using REST clients such as [Postman](https://www.postman.com/) or a dedicated SPARQL library such as
[rdflib](https://rdflib.readthedocs.io/en/stable/) or [Apache Jena](https://jena.apache.org/).

```bash
curl -X POST -H "Content-Type: application/sparql-query" --data "SELECT * WHERE { ?s ?p ?o } LIMIT 10" http://localhost:3030/jel/sparql

```

```sql
QUERY
=
$(cat docs/construct_example.sparql | tr -d '\n')
ENCODED_QUERY
=
$(python -c "import urllib.parse; print(urllib.parse.quote('''$QUERY''')
)
")
curl "
http
://
localhost
:
3030/
jel/
sparql
?
query
=
$ENCODED_QUERY
"
```
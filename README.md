# SmartKG+ Server
This repository contains the server implementation of the WiseKG as an extension of the TPF server.

# Linked Data Fragments Server <img src="http://linkeddatafragments.org/images/logo.svg" width="100" align="right" alt="" />
On today's Web, Linked Data is published in different ways,
which include [data dumps](http://downloads.dbpedia.org/3.9/en/),
[subject pages](http://dbpedia.org/page/Linked_data),
and [results of SPARQL queries](http://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=CONSTRUCT+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D%0D%0AWHERE+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D&format=text%2Fturtle).
We call each such part a [**Linked Data Fragment**](http://linkeddatafragments.org/).

The issue with the current Linked Data Fragments
is that they are either so powerful that their servers suffer from low availability rates
([as is the case with SPARQL](http://sw.deri.org/~aidanh/docs/epmonitorISWC.pdf)),
or either don't allow efficient querying.

Instead, this server offers **[Triple Pattern Fragments](http://www.hydra-cg.com/spec/latest/triple-pattern-fragments/)**.
Each Triple Pattern Fragment offers:

- **data** that corresponds to a _triple pattern_
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=rdf%3Atype&object=dbpedia-owl%3ARestaurant))_.
- **metadata** that consists of the (approximate) total triple count
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=rdf%3Atype&object=))_.
- **controls** that lead to all other fragments of the same dataset
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=&object=%22John%22%40en))_.

This is a **Java** implementation based on Jena. 

NOTE: the following commands were executed using Java 21.

## Build
Execute the following command to create a WAR and JAR file:
```
$ mvn install
```
## Deploy stand alone
The server can run with Jetty from a single jar as follows:

```
java -jar target/ldf-server.jar [config.json]
```

The `config.json` parameters is optional and is default the `config-example.json` file in the same directory as `target/ldf-server.jar`.

## Deploy on an application server
Use an application server such as [Tomcat](http://tomcat.apache.org/) to deploy the WAR file.

Create an `config.json` configuration file with the data sources (analogous to the example file) and add the following init parameter to `web.xml`:

    <init-param>
      <param-name>configFile</param-name>
      <param-value>path/to/config/file</param-value>
    </init-param>
  
If no parameter is set, it looks for a default `config-example.json` in the folder of the deployed WAR file.

## Status
This software is still under development. It currently supports:
- HDT & Jena TDB data sources
- HTML, Turtle, NTriples, JsonLD, RDF/XML output

A [more complete server](https://github.com/LinkedDataFragments/Server.js/) has been implemented for the Node.js platform.


### Usage

To access the server, the following endpoints are available:

- `http://localhost:8080/smartkg+`: the original data source in HDT format. This should be used for executing queries on the original data, and for executing queries that cannot be executed on the partitioned data or when no partition can be selected for a query. `smartkg+` could be replaced with whatever name you have given to the HDT data source in the configuration file. An example of a query looks like this: `http://localhost:8080/smartkg+?subject=&predicate=&object=`
- `http://localhost:8080/molecule/smartkg+`: returns the metadata of the partitioned data. It should be noted when your data contains lots of partitions, the metadata file can be quite large, and it might take a while to load the metadata.
- `http://localhost:8080/molecule/smartkg+/2.hdt`: returns the partition with id 2. The name of the partition should be the same as the name of the partition files in the metadata file. It should be noted that if the `numTriples` field is 0, the partition is empty and no file will be returned, as it does not exist.
- `http://localhost:8080/partition/2`: returns the same partition with id 2, but in turtle format instead of HDT. The name of the partition should be the same as the name of the partition files in the metadata file.
- `http://localhost:8080/plan?bgp=...`: returns the execution plan for the given BGP query. The BGP query should be URL encoded. The most baseline example of a BGP query looks like this: `http://localhost:8080/plan?bgp=%3Fs%20%3Fp%20%3Fo%20.`, which just returns the plan for all possible triples. Additional filters can be added to the BGP query, for example: `http://localhost:8080/plan?bgp=%3Fs%20%3Fp%20%3Fo%20.&speed=1&latency=1000&"`. The `speed` parameter is used to indicate the speed of the network connection in Mbps, and the `latency` parameter is used to indicate the latency of the network connection in ms.

### Config file
A small note on the configuration file. An example is also provided and was used in the benchmarking of the WiseKG: `config.json`. The following parameters are used:
- `metadatapath` is the path to the metadata file of the partitioned data. It should be noted that the metadata file should be in JSON format.
- `moleculesdatapath` is the path to the folder that contains the partition files in HDT format. The name of the partition files should be the same as the name of the partition files in the metadata file.
- `cspath` is the path to the file of the original HDT data source but with a `.cs` extension. ++++
- `partstring` is the pre-string for the partition files. For example, if the partition files are named `partition_1.hdt`, `partition_2.hdt`, etc., then the `partstring` should be `partition_`. If none was provided, you can set this value to an empty string.
- `uri` and `default`.....
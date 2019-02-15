# Faceted Search for the VIVO community
Faceted Search for VIVO using ElasticSearch
This works with ElasticSearch 6.4.1
It has been tested against VIVO 1.9.3
A working example can be seen at: https://experts.colorado.edu/publications

## Overview

To configure a working faceted browser you need:
1. A running instance of ElasticSearch with a populated index. It has been vetted with ES 6.4.1
2. A webpage with references to facetview2 scripts and an embedded configuration
 * the facetview2 scripts are a submodule under the html directory


You can find out more about ElasticSearch at [https://www.elastic.co/products/elasticsearch](https://www.elastic.co/products/elasticsearch)

## Installation steps

### ElasticSearch Installation

Instructions for setting up a local instance of ElasticSearch are available at [https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)


### The Ingest Process

Currently we use python scripts to build and import batch documents into our ElasticSearch (ES) service.  The scripts can be scheduled or run manually and currently generate a batch of all documents for a given search type (e.g. publication, person, dataset, etc).  Our scripts do not currently support generating batches of only recently changed documents.  The ingest scripts the VIVO sparqlQueryAPI endpoint and generate a ES batch file containing JSON documents for all entities to be indexed.

Ingest scripts are located in the ingest directory and follow the convention rebuild-x.sh  where x the plural form of the 'type' of search document generated. For example rebuild-pubs.sh. This calls a python script called ingest-publications.py

Currently a SELECT query is run to get a list of all URIs to build search documents for and then construct queries are run for each URI.  
Our current publication ingest takes about 30 minutes to run, it generate about 87K publications, the actual import into ES usually takes a couple seconds.

### Importing data into ElasticSearch

ElasticSearch (ES) is a search server based on Lucene. It provides a distributed, multitenant-capable full-text search engine with a RESTful web interface and schema-free JSON documents.  

ES is capable of indexing nested JSON documents.  This is a very useful feature as it makes it easy to build a JSON document containing nested objects that can be used to for search retrieval and faceting.

Documentation on bulk import of ES documents is available at [https://www.elastic.co/guide/en/elasticsearch/reference/current/_batch_processing.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/_batch_processing.html).

#### Custom mapping import

ES will auto-generate a schema for an imported document if no mappings currently exist.  If a mapping already exists it will attempt to update the mapping with definitions for any fields from the imported document that did not exist in the current mapping.  If there is a conflict with an imported document and the current mapping the service will return an error message.

We have found it useful and necessary to customize mappings for values used in facets.  See the mapping JSON documents in ingest/mappings for examples.

Documentation on ES schemas and how a mapping can be customized are available at [https://www.elastic.co/blog/found-elasticsearch-mapping-introduction](https://www.elastic.co/blog/found-elasticsearch-mapping-introduction)

The ingest scripts will push a customized mapping during the publish phase if --mapping is specified with a path to the mapping file.

### Setting up the faceted browser in VIVO

Embedding a faceted browser in a webpage is very simple.

An html/publications.ftl page is provided to allow you to bootstrap a VIVO page with your publicaitons.
This needs to be placed into your VIVO directory under your theme.
For example we place it in our VIVO 3rd tier at: 
  ./webapp/src/main/webapp/themes/cu-boulder/templates/publications.ftl

Next the facetview2 software needs to be placed in your VIVO installation.
For example we placed it in our VIVO 3rd tier at:
  ./webapp/src/main/webapp/themes/cu-boulder/facetview2

Finally you need to create a self contained page in your VIVO that uses the publications.ftl template. 

Thats it!!

If you want to see the details of how facetview2 is used just look at the html/publications.ftl document.

## Background
This was introduced to the VIVO community by Stephan Zednik who worked with Deep Carbon Observatory and RPI to create a simple faceted browser for VIVO. The code leveraged work by Cottage Labs: https://github.com/CottageLabs/facetview2

Benjamin Gross has also done a significant amount of work to adapt the code for UNAVCO eg (https://connect.unavco.org/datasets).
This led to modifications which allowed the ingest scripts to work against the sparqlQueryAPI instead of Fuseki.

Don Elsborg has worked on this at CU Boulder eg (https://experts.colorado.edu/publications) to develop more comprehensive views and facets on publications. This supported a publications page which showed and faceted on organizations and research areas. 
Additionally the original facetedsearch developed by DCO and Cottage Labs did not work on Elastic versions greater or equal to 5. 

## Todo
1. Potentially work with Cottage Labs to merge the 6.4.1 branch into their repository
2. Use the ingest tooling to create a production evolution ES document
3. Add a people.ftl page

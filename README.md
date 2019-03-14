# AmbiverseNLU Kubernetes Deployment
[![GitHub license](https://img.shields.io/github/license/linuskohl/ambiverse-nlu-kubernetes.svg?style=popout-square)](https://github.com/linuskohl/ambiverse-nlu-kubernetes)

## Synopsis
AmbiverseNLU is an amazing natural language understanding suite built by Ambiverse and the Max-Planck-Institute.
This repository contains the files to deploy [AmbiverseNLU](https://github.com/ambiverse-nlu) in the (Google) Kubernetes. 
There are two main services. 
- Ambiverse NLU, (requires Apache Cassandra)
- Knowledge Graph Service (requires Neo4j)

This should be considered for testing purposes only. For production you should build your Cassandra and Neo4j clusters first and then import the data.
I am not affiliated with neighter Ambiverse or the Max-Planck-Institute, who built this great piece.


## Requirements
You should not be to scarce with your K8s cluster node configuration, specially in concern of memory.
The [documentation](https://github.com/ambiverse-nlu/ambiverse-nlu/blob/master/README.md) of the NLU service suggests at least 16Gb of main memory - per document you want to process simultaneously. Think 40Gb instead of 8 ;-) 

## Installation
Clone this repository anywhere by issuing
```bash
git clone https://github.com/linuskohl/ambiverse-nlu-kubernetes
```
## Configuration

## Deployment
```bash
kubectl apply -f 00-namespaces.yml -f 10-config.yml 
kubectl apply -f 20-cassandra-service.yml -f 21-neo4j-service.yml -f 22-nlu-service.yml -f 23-knowledgegraph-service.yml
kubectl apply -f 30-cassandra-statefulset.yml
kubectl apply -f 31-neo4j-statefulset.yml
```
After neo4j has successfully started, you need to enter your neo4j container
```bash
kubectl --namespace=nlu-ambiverse exec -it neo4j-0 /bin/bash
```
and run the following commands
```bash
cypher-shell
CREATE INDEX ON :WikidataInstance(url);
CREATE INDEX ON :Location(location);
CALL apoc.periodic.commit("MATCH (l:Location) where not exists(l.location) with l limit 10000 SET l.location =  point({latitude: l.latitude, longitude: l.longitude, crs: 'WGS-84'}) return count(l)", {});
```
Now you can deploy the main applications
```bash
kubectl apply -f 32-knowledgegraph-deployment.yml -f 33-nlu-deployment.yml
```

## Test Setup
### AmbiverseNLU
```bash
curl --request POST \
  --url http://LOAD_BALANCER_IP:8080/factextraction/analyze \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --data '{"docId": "doc1", "text": "Jack founded Alibaba with investments from SoftBank and Goldman.", "extractConcepts": "true" }'
```
###  Knowledge Graph Service
```bash
curl -X POST -H "Content-Type: application/json" \
      -d '[
            "http://www.wikidata.org/entity/Q1137062",
            "http://www.wikidata.org/entity/Q1359568"
          ]' \
      "http://LOAD_BALANCER_IP:8080/knowledgegraph/entities"
```

### License
The project is licensed under the MIT License. See [LICENSE.txt](https://github.com/linuskohl/ambiverse-nlu-kubernetes/blob/master/LICENSE.txt) for more details.

### Author
- Linus Kohl, linus@munichresearch.com

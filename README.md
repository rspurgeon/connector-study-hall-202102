# connector-study-hall-202102
Bleeps and bloops from the Feb 21 2021 meetup on managed connectors and DevOps

# Tech

* Kafka Connect
  * [From Zero to Hero with Kafka Connect](https://talks.rmoff.net/ScGJTe#sjBYBqW) credit: [Robin Moffatt](https://github.com/rmoff)
	* [Confluent Cloud Managed Connector Docs](https://docs.confluent.io/cloud/current/connectors/index.html)

* Confluent Cloud 
  * [New Signups](https://confluent.cloud/signup)
  * Use the Coupon Code `CONNECT200` for a $200 credit
	* [Confluent Cloud CLI](https://docs.confluent.io/ccloud-cli/current/index.html)

* jq: command-line JSON processor
	* [Download](https://stedolan.github.io/jq/download/)

* Google Cloud Platform
  * [Signup](https://console.cloud.google.com/freetrial/signup/tos)
  * [Google Big Query](https://cloud.google.com/bigquery)
	* [Confluent BigQuery Sink connector docs](https://docs.confluent.io/cloud/current/connectors/cc-gcp-bigquery-sink.html)

* streaming-ops
  * [GitHub](https://github.com/confluentinc/streaming-ops)
	* [Docs](https://docs.confluent.io/platform/current/tutorials/streaming-ops/index.html)
  * [Blog: DevOps for Apache Kafka with Kubernetes and GitOps](https://www.confluent.io/blog/devops-for-apache-kafka-with-kubernetes-and-gitops/)
  * [Blog: Spring Your Microservices into Production with Kubernetes and GitOps](https://www.confluent.io/blog/spring-microservices-into-production-with-kubernetes-gitops/)

# Workshop Command Reference

* Login to ccloud
```
ccloud login --save
```

* To set the current ccloud environment you want to work with:
```
ccloud environment list

ccloud environment use env-pwv35
Now using "env-pwv35" as the default (active) environment.
```

* To set the current kafka cluster you want to work with:
```
ccloud kafka cluster list

ccloud kafka cluster use lkc-36r20
Set Kafka cluster "lkc-36r20" as the active cluster for environment "env-pwv35".
```

* We need an API Key to auth with Kafka, use ccloud api-create to 
  store it in a file that won't get added to source control. For prod use cases use a key assigned to a service
	account with ACLs applied [Docs](https://docs.confluent.io/cloud/current/access-management/acl.html).
```
cat api-key-example.json

ccloud api-key create --resource lkc-36r20 -o json > .secret/kafka-api-key.json
```

* We need an API Key to auth to the Confluent Cloud control plane, same routine as above
```
ccloud api-key create --resource cloud -o json > .secret/api-key.json
```

* Using `jq`, put the Kafka API Key and Secret into env vars so we can template them into the connector configuration before POSTing
```
export KAFKA_API_KEY=$(jq -r '.key' .secret/kafka-api-key.json)
export KAFKA_API_SECRET=$(jq -r '.secret' .secret/kafka-api-key.json)
export CURL_USER=$(jq -r '.key' .secret/api-key.json)
export CURL_PWD=$(jq -r '.secret' .secret/api-key.json)
```

* GCP automates auth with a "keyfile" in the form of JSON document.  In order to post 
  this keyfile to a connector, we need to "stringify it so the full managed connector doc 
	remains a valid JSON doc. To "Stringify" the GCP Keyfile, we use `jq`.
```
cat bq-keyfile-example.json

export ORDERS_BQ_SINK_KEYFILE=$(cat .secret/devx-gcp-keyfile.json | jq -r -c)
```

* Template the environmnet variables into a JSON document we can post to the connector REST API, 
	and the result will look like the example bq-sink-post-example.json
```
cat bq-sink-template.json

jq -n -f ./bq-sink-template.json > .secret/bq-sink-post.json

cat bq-sink-post-example.json
```

* Now we use the Confluent Cloud Connector REST API to post the connector to Confluent Cloud 
  with our environment and cluster ID in the URL
```
curl -XGET -H 'Content-Type: application/json' --user "$CURL_USER:$CURL_PWD" https://api.confluent.cloud/connect/v1/environments/env-pwv35/clusters/lkc-36r20/connectors

curl -XPOST -H 'Content-Type: application/json' --data "@.secret/bq-sink-post.json" --user "$CURL_USER:$CURL_PWD" https://api.confluent.cloud/connect/v1/environments/env-pwv35/clusters/lkc-36r20/connectors
```

* Now let's do the same operation with the ccloud CLI.  Currently, the CLI requires
  only the configuration portion of the connector definition. So we pull that out 
  once again with `jq` and we'll update the name so we get a second connector:
```
jq -r '.config' .secret/bq-sink-post.json | jq '.name = "orders-bq-sink-ccloud' > .secret/bq-sink-ccloud.json

ccloud connector create --config .secret/bq-sink-ccloud.json
```


# connector-study-hall-202102
Bleeps and bloops from the Feb 21 2021 meetup on managed connectors and DevOps

# Tech

* Confluent Cloud 
  * New Signups: https://confluent.cloud/signup
  * Use the Coupon Code `CONNECT200` for a $200 credit
	* [Confluent Cloud CLI](https://docs.confluent.io/ccloud-cli/current/index.html)

* jq: command-line JSON processor
	* https://stedolan.github.io/jq/download/

* Google Cloud Platform
  * https://console.cloud.google.com/freetrial/signup/tos
  * [Google Big Query](https://cloud.google.com/bigquery)

# Command Reference

* To set the current ccloud environment you want to work with:
```
ccloud environment use env-pwv35
Now using "env-pwv35" as the default (active) environment.
```
* To set the current kafka cluster you want to work with:
```
ccloud kafka cluster use lkc-36r20
Set Kafka cluster "lkc-36r20" as the active cluster for environment "env-pwv35".
```
* We need an API Key to auth with Kafka, we can take a shortcut and create a "SuperUser" key for the time being
  and store it in a file that won't get added to source control
```
ccloud api-key create --resource cloud -o json > .secret/api-key.json
```



# References

* streaming-ops
  * [GitHub](https://github.com/confluentinc/streaming-ops)
	* [Docs](https://docs.confluent.io/platform/current/tutorials/streaming-ops/index.html)
  * [Blog: DevOps for Apache Kafka with Kubernetes and GitOps](https://www.confluent.io/blog/devops-for-apache-kafka-with-kubernetes-and-gitops/)
  * [Blog: Spring Your Microservices into Production with Kubernetes and GitOps](https://www.confluent.io/blog/spring-microservices-into-production-with-kubernetes-gitops/)


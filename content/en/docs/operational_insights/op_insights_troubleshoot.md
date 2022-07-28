---
title: Troubleshooting Operational Insights
linkTitle: Troubleshoot
weight: 300
date: 2022-06-09
description: Troubleshoot problems you might encounter when configuring and running Operational Insights.
---

## Solution does not show data in real time

An important aspect of this capability is to store the API requests as fast as possible in Elasticsearch, so that they are available in real-time in the API Gateway Traffic monitor or in the Kibana dashboard.

If your setup cannot process the log information fast enough, then the solution may never be real-time and eventually even log information may be lost.

Watch [Axway API-Management Elasticsearch Integration - Performance Troubleshooting](https://www.youtube.com/watch?v=rLT84dkgGAE) video to learn why the document ingest rate might be too low, what information is available for analysis in Kibana, and how to correct this if necessary.

## Confirm that processes or containers are running

From within your GitHub project's folder where the `docker-compose.yml` file is located, run the following:

```bash
docker-compose inspect
```

The status of each container is shown depending on the services you enabled and disabled.

|                              Name                                 |            Command            |State        |Ports|
|---                                                                |---                            |---          |---  |
|apigateway-openlogging-elk_elk-traffic-monitor-api_1_3fbba4deea37  |docker-entrypoint.sh node     |Up (healthy)  |0.0.0.0:8889->8080/tcp|
|apigateway-openlogging-elk_filebeat_1_3ad3117a1312                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9000->9000/tcp|
|apigateway-openlogging-elk_logstash_1_c6227859a9a4                 |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:5044->5044/tcp, 9600/tcp|
|elasticsearch1                                                     |/usr/local/bin/docker-entr ...|Up            |0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp|

## Verify Filebeat is receiving data

You need to check the filebeat Log-File within the running docker container.

```bash
docker exec -it apigateway-openlogging-elk_filebeat_1_3ad3117a1312 bash
cd logs
tail -f filebeat
```

Verify the Filebeat harvester is started on the Open Traffic files:

```bash
INFO    log/harvester.go:251    Harvester started for file: /var/log/work/group-2_instance-1_traffic.log
```

The following error means that Logstash is not running or reachable, or just not yet fully started:

```bash
ERROR   pipeline/output.go:100  Failed to connect to backoff(async(tcp://logstash:5044)): lookup logstash on 127.0.0.11:53: no such host
```

{{< alert title="Note">}}
Filebeat does not shows a status when it is successfully processing your log files.
{{< /alert >}}

If you do not see any errors after the harvester process is started, you can assume your files are processed. If the harvester does not start, you might need to validate that the Open Traffic Event log files are visible by listing the following directory within the running container:

```bash
ls -l /var/log/work
-rw-rw-r--. 1 filebeat filebeat  2941509 Aug 13 12:38 group-2_instance-1_traffic.log
-rw-rw-r--. 1 filebeat filebeat 20972249 Jul  7 19:32 group-2_instance-1_traffic_2020-07-07-1.log
-rw-rw-r--. 1 filebeat filebeat 20972436 Jul  8 18:08 group-2_instance-1_traffic_2020-07-08-1.log
-rw-rw-r--. 1 filebeat filebeat 20972005 Jul 17 07:32 group-2_instance-1_traffic_2020-07-17-1.log
```

## Verify Logstash processing

Logstash writes to stdout, so you can view some log information just by running the following:

```bash
docker logs logstash -f
```

After Logstash is successfully started, you should see the following result:

```bash
docker logs logstash
Pipelines running {:count=>6, :running_pipelines=>[:".monitoring-logstash", :DomainAudit, :Events, :TraceMessages, :BeatsInput, :OpenTraffic], :non_running_pipelines=>[]}
Successfully started Logstash API endpoint {:port=>9600}
```

If you see the following or similar error message during processing of events, then API Builder lookup API cannot be reached. In this case, ensure the environment variable  `API_BUILDER_URL` is set correctly.

```bash
[2020-08-19T10:51:05,671][ERROR][logstash.filters.http    ][main][......] error during HTTP request {:url=>"/api/elk/v1/api/lookup/api", :body=>nil, :client_error=>"Target host is not specified"}
```

## Verify Admin node manager is running

If the Admin node manager (ANM) does not fetch the data from Elasticsearch by way of API Builder, it is recommended to check the ANM trace file. Possibly errors like the following are logged in the trace file:

```bash
ERROR   07/Dec/2020:01:43:19.769 [262e:27ebcd5f0601b4bda7f620b5]         java exception: 
java.lang.RuntimeException: java.net.URISyntaxException: Illegal character in path at index 0: [invalid field]/api/elk/v1/api/router/service/instance-1/ops/search?format=json&field=leg&value=0&count=1000&ago=24h&protocol=http
```

In this case, the environment variable `API_BUILDER_URL` is not set for the ANM process.

It can also be useful to use the Developer tools (Network monitor) in your browser (for example, Chrome or Firefox) to check the result of API requests in the Traffic Monitor. This way you can see the error straight away when API Builder fail to perform the request.

The following animations shows how to use the Developer tools:

* **add the gif here**

## Verify Elasticsearch processing

It takes a while until Elasticsearch is started. Run the following command to monitor when the process started:

```bash
docker logs elasticsearch1 --follow
```

When Elasticsearch is started you should see the following message:

```
"level": "INFO", "component": "o.e.c.r.a.AllocationService", "cluster.name": "elasticsearch", "node.name": "elasticsearch1", "message": "Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[.kibana_1][0]]]).", "cluster.uuid": "k22kMiq4R12I7BSTD87n5Q", "node.id": "6TVkdA-YR7epgV39dZNG2g"  }
```

Status YELLOW is expected when running Elasticsearch on a single node, as it cannot achieve the desired replicas. You might want to use Kibana Development tools or CURL to get additional information.

## Virtual memory is too low

The following error is show when virtual memory is too low:

```bash
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

Run the following to increase it temporarily:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Alternatively, you can also run:

```bash
sudo vi /etc/sysctl.conf
add vm.max_map_count=262144
sudo service sysctl restart
```

## No results from Elasticsearch

If you don't get any results from Elasticsearch for valid queries, an index template might not be applied correctly during index creation.

Be aware that Elasticsearch does not execute queries on the original document, rather on the indexed fields. How these were indexed is defined by an index mapping. For this purpose, this component delivers an index template for each index, which is used when the index is created.

You can find the index mapping in the API Builder container, `elasticsearch_config/<index-name>/index_template.json` or you can download it [here](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk/tree/develop/apibuilder4elastic/elasticsearch_config).

To check wether the index mapping was applied correctly to an index, execute the following request. For example the Traffic-Summary index:

```bash
http://elasticsearch:9200/apigw-traffic-summary/_mapping
```

Then check whether properties mappings defined in the configuration mentioned above have been applied to the index.

## Monitor API Builder processing

To verify that API Builder docker container is running, run:

```bash
docker logs apibuilder4elastic --follow
```

If the container is running you should see the following message:

```bash
docker logs apibuilder4elastic --follow
```

## Verify requests from Admin node manager

When using the API-Gateway Traffic-Monitor to monitor requests and having the Admin-Node-Manager re-configured you should see how API-Builder is processing the requests:

```bash
Request {"method":"GET","url":"/api/elk/v1/api/router/service/instance-1/ops/search?format=json&field=leg&value=0&count=1000&ago=10m&protocol=http","headers":{"host":"localhost:8889","max-forwards":"20","via":"1.0 api-env (Gateway)","accept":"application/json","accept-language":"en-US,en;q=0.5","cookie":"cookie_pressed_153=false; t3-admin-tour-firstshow=1; VIDUSR=1584691147-TE1M3vI9BFWgkA%3d%3d; layout_type=table; portal.logintypesso=false; portal.demo=off; portal.isgridSortIgnoreCase=on; 6e7e1bb1dd446d4cd36889414ccb4cb7=8g9p3kh27t1se22lu6avkmu0a1; joomla_user_state=logged_in; 220b750abfbc8d2f2f878161bab0ab65=62gr71dkre858nc0gjldri18gt","csrf-token":"8E96374767C47BFADC9C606FF969D7CF56FB3F9523E41B34F3B3B269F7302646","referer":"https://api-env:8090/","user-agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/74.0","x-requested-with":"XMLHttpRequest","connection":"close","x-correlationid":"Id-fd7c745ebfaed039b2155481 1"},"remoteAddress":"::ffff:172.25.0.1","remotePort":55916}

Response {"statusCode":200,"headers":{"server":"API Builder/4.25.0","request-id":"35fb859d-00b0-404b-97e6-b549db17f84c","x-xss-protection":"1; mode=block","x-frame-options":"DENY","surrogate-control":"no-store","cache-control":"no-store, no-cache, must-revalidate, proxy-revalidate","pragma":"no-cache","expires":"0","x-content-type-options":"nosniff","start-time":"1584692477587","content-type":"application/json; charset=utf-8","response-time":"408","content-md5":"e306ea2d930a3b80f0e91a29131d520b","content-length":"267","etag":"W/\"10b-2N+JsHuxDxMVKhJR1A8GuNGnKDQ\"","vary":"Accept-Encoding"}}
```

If you do not see any requests arriving in the API builder, the ANM may not be able to reach the API builder listen socket. It is important to know that traffic information will still appear in this case, because in this case the OPSDB will be used. You should therefore check the ANM trace log.

```bash
tail -f /opt/Axway/APIM/apigateway/trace/nodemanageronapi-env_20200813000000.trc
```

## Check queries send to ElasticSearch

In oder to see queries that are send to ElasticSearch by API-Builder you need to run the Docker-Container with LOG_LEVEL=debug. You can activate debug in the docker-compose.yml. This gives you in the console of the API-Builder the following output:

```bash
Using elastic search query body: {"index":"logstash-openlog","body":{"query":{"bool":{"must":[{"range":{"timestampOriginal":{"gt":1587541496568}}},{"term":{"processInfo.serviceId":"instance-1"}}]}}},"size":"1000","sort":""}
```

This helps you to further analyze if ElasticSearch is returning the correct information for instance using the Kibana development console. Example sending the same request using the Kibana Development console:

(**there's an image here but it would be better to add the code, instead of the screenshot of the code**)

## ILM Rollover alias error

If the solution is configured with different regions, you may see the following error in Kibana or in the Elasticsearch logs:

```bash
index.lifecycle.rollover_alias [apigw-traffic-summary] does not point to index [apigw-traffic-summary-us-dc1-000002]
```

When an index is rolled, for example because the configured size, time has been reached or due to an update, a new index is created based on the corresponding index template. In this template the ILM rollover alias is configured, which does not match the index if it is regional. The API Builder checks periodically (every 10 minutes) if the ILM rollover alias is correct and adjusts it if necessary. You can also start this modification manually:

```bash
docker exec apibuilder4elastic wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/index/rolloverAlias -O/dev/null
```

Additionally you may set the index rollover alias to the correct value using for example this request:

```bash
PUT /apigw-trace-messages-eu-000001/_settings
{
  "index" : {
    "lifecycle.rollover_alias" : "apigw-trace-messages-eu"
  }
}
```

## Check Caching

The solution uses Memcache in API Builder & Logstash to avoid unnecessary duplicate queries for APIs and users. To check if entries are cached correctly, you can connect to the memcached via telnet localhost 11211. Some examples how to use it.

### Get slabs

Memcache is storing the data into slabs, which are like chunks depending

```bash
stats cachedump 2 0
ITEM ignoredAPIs:###Health Check [19 b; 1610535266 s]
ITEM ignoredAPIs:/healthcheck### [19 b; 1610535266 s]
ITEM index_status:events###N/A [27 b; 1610538266 s]
ITEM index_status:trace###N/A [27 b; 1610538266 s]
ITEM index_status:openlog###N/A [27 b; 1610538266 s]
END
```

You may also try `stats cachedump 1 0` instead. The example above shows that the index_stat

To get a key, for instance the version check status:

```bash
get version_status:versionCheck
VALUE version_status:versionCheck 1 89
{I"
   message:ETI"'Filebeat and Logstash version okay;TI"versionStatus;TI"ok;T
END
```

You can find additional information here: <https://techleader.pro/a/90-Accessing-Memcached-from-the-command-line>. You may also use [PHPMemcachedAdmin](https://github.com/elijaa/phpmemcachedadmin) to get insights about the Memcache instance.

## Certificate error Admin Node Manager to API Builder

If you have errors connecting from the Admin Node Manager to the API Builder, then the following instructions: Please make sure that the Admin-Node Manager has imported the correct CA.
If you connect to the API Builder using a hostname other than `apibuilder4elastic`, you may receive the following error message:

```
The certificate was not issued by the domain issuer
```

To solve the problem you can create a [remote host](https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_policydev/apigw_gw_instances/general_remote_hosts/index.html) and disable the hostname validation (`Verify server’s certificate matches requested hostname`). Or you can use your own matching keys & [certificates](https://github.com/Axway-API-Management-Plus/apigateway-openlogging-elk#custom-certificates).

Another reason might be a missing or expired license for the Admin-Node-Manager. Please see <https://support.axway.com/kb/178766/language/en>.

## Filebeat - Failure to publish events

If you see the following error message in the filebeat log:

```
failed to publish events: write tcp 172.23.0.2:55204->172.31.48.67:5044: write: connection reset by peer
```

This means that Logstash could not accept any more events. This does not automatically mean that Logstash instances have to be added, but can also mean that the Elasticsearch cluster can not process any more events.
The following options:

* Check the CPU utilization of the Logstash nodes. Restart them if necessary.
* It sounds crazy, but it can help not to specify the logstash hosts in the same order on every filebeat.
* Check that the Logstash instances have configured all available Elasticsearch nodes.
* Add more Logstash nodes on a test basis, if that doesn't improve things, add an Elasticsearch node.

## Kibana - Missing Long-Term statistics

If the Quarterly and Yearly API request processing dashboards do not display any data, check if the transform job: `apigw-traffic-summary-hourly-v<n>` is running and started. The API-Builder4Elastic application tries to create this job every hour. You can also start this with the following API request in the API-Builder container:

```bash
docker exec apibuilder4elastic wget --no-check-certificate https://localhost:8443/api/elk/v1/api/setup/transform/apigw-traffic-summary
```

If the job exists, check if the index: `apigw-hourly-traffic-summary-00000<n>` exists. If not, please stop the transform job and start it again. If necessary check the messages of the transform job. If the index exists but does not contain any documents, please delete it and restart the transform job.
Finally you can check if the index pattern: `apigw-hourly-traffic-summary*` exists. If not, please re-import the Kibana dashboard configuration: `Axway-api-overview.ndjson`.

If the **API status history** and **API Gateway status history** in the Quarterly or Yearly-Dashboards are not shown, perform the following.

* Stop and Delete the Transform-Job: `apigw-traffic-summary-hourly-v<n>`
* Delete the hourly index: `apigw-hourly-traffic-summary-00000<n>`

Then, either wait for max. 60 minutes until API-Builder has re-created the transform or call the REST-API as shown above

The main cause of this problem is the way the http.status field is indexed, which was changed in the update to version 3.4.0. If no documents have been received after the upgrade and initial creation of the transformation job, the transformation does not correctly index the http.status.keyword field.

## Too many API-Manager lookups

API- and Application-Details are looked up as part of the Logstash pipeline from the API Manager using a APIBuilder4Elastic REST API. The obtained details are cached by Logstash using Memcache.
Now, if lookups are constantly performed nevertheless, then this might be caused by APIs which have a path parameter (e.g /api/v1/cusomer/323213).
Because the path parameter makes the API path dynamic it prevents Logstash from caching it. In this case, you need to help the solution by setting up the parameter: `CACHE_API_PATHS`.
You may also watch the following video to learn more about this topic: <https://youtu.be/rLT84dkgGAE>

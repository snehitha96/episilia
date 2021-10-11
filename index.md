## **EPISILIA**
### **Prerequisites**:
The following prerequisites are required to install Episilia.
```
1. Installing and configuring Helm (Helm CLI)
2. Kubectl CLI
3. A Kubernetes Cluster Up and Running
4. Setup Kafka (kafka IP address should be mentioned in Episilia)
      Topics:
         - Episilia-logs(kafka topic where logs are pushed)
         - Episilia_index_1(internal live topic)
         - Episilia_opt_1(internal optimized topic)
         - episilia-metrics
         - Episilia_opt_req	(internal optimizer request topic)
      The Logs to be pushed to this topic “episilia-logs” and only this topic can be overridden in the charts and others are   default internal topics.
5. Docker login (login into DockerHub)
6. Create Secret (To pull the images using secret, use regcred as default secret name) 
   kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/Deskstop/.docker/config.json --type=kubernetes.io/  dockerconfigjson
7. S3 credentials where the Index and data files to be stored (aws accesskey, secret key, bucket name, folder name, region)
```

#### **Note:**
1. Links for Reference of the Prerequisites are given at the end of the Doc.
2. Refer to the following link for usage of episilia for the end users to search of logs using grafana GUI.
   [help](https://gitlab.com/episilia-public/episilia-install/-/blob/master/EpisiliaUserGuide.pdf)

#### **Step1: Adding the helm repo to local Machine**
 Repo URL is Gitlab Page URL
 ```bash
helm repo add [NAME] [URL]
$ helm repo add episilia https://episilia.gitlab.io/episilia-helm/release
```
#### **Listing chart repositories:**
```
$ helm repo list or helm repo ls
  NAME        URL 
  episilia   https://episilia.gitlab.io/episilia-helm/release
```
#### **Searching for charts in the repository:**
```
$ helm search repo episilia
```
#### **Step2: In above picture episilia/episilia-cpanel is the master chart. In this master chart's values.yaml file update kafka values and  required global values**
Inspect the values before installing application use below:
```
$ helm inspect values episilia/episilia-cpanel > episilia_values.yaml
```
 Update values file with your values in the global field and  We can pass values file (episilia.values) while installing the application.
The global values are :

| Sl.No | Key | Value |
| ------ | ------ | ------ |
|     **AWS Configurations**   |
| 1 | datastore.s3.accesskey | AWS access key |
| 2 | datastore.s3.secretkey | AWS secret key |
| 3 | datastore.s3.region    | AWS region     |
| 4 | datastore.s3.bucket    | AWS S3  Bucket name(this has to be same on what search is configured) |
| 5 | datastore.s3.folder    | Folder name to be created inside the bucket |
| 6 | datastore.s3.work.folder | Folder name to store s3 limit/write results|
| 7 | datastore.s3.endpoint.url| To be changed if using any other storage service than AWS s3 |
| 8 | datastore.s3.sign.payload|Default true (not to be changed) |
| 9 | datastore.s3.url.prefix| To be changed if using any other storage service than AWS s3 |
|                        **Kafka log_consumer Event listeners**                                      |
| 10| kafka.metadata.broker.list | Kafka endpoint with port no. |
| 11| kafka.group.search    | Kafka consumer-group for search |
| 12|   kafka.group.cpanel  |  Kafka consumer-group for cpanel|
| 13|   kafka.topic.index.live  |Topic for publishing indexed files (internal) |
| 14|  kafka.topic.index.optimized  | Topic for optimizing of files (internal) |
| 15| kafka.topic.optimize.request    | Topic for optimizer to take request  for optimizing files (internal) |
| 16| kafka.topic.cpanel.in | Kafka topic name used by servers to update certain config(currently not being used) |
| 17| kafka.topic.cpanel.out    | Kafka topic name to which server metrics are pushed for cpanel |
| 18|  kafka.indexer.logs.topics  |Kafka topic name to which logs will be pushed |
| 19|  kafka.indexer.group   | Kafka consumer-group for indexer |
|           **Indexer**        |
| 20|  indexer.schema.appid.fixed   | If appid is a fixed string |
| 21| indexer.schema.appid.keys    | label(s) for app identifier |
| 22| indexer.schema.tenantid.fixed    | If tenantid is a fixed string |
| 23| indexer.schema.tenantid.keys    | label(s) for tenant identifier  |
| 24|  indexer.schema.message.key   | actual log message key  |
| 25| indexer.schema.timestamp.key    | timestamp key |
| 26|  indexer.schema.timestamp.formats  | to specify timestamp format (ex: %Y-%m-%dT%H:%M:%S ) |
| 27| indexer.schema.exclude    |  |
|       **Search**          |
| 28|   search.api.timeout.seconds  | timeout for search while querying |
| 29| search.mode |  default is ‘live’, can be either ‘live’ or ‘fixed,|
| 30|  search.live.from.hours  | hours from when the required index blocks should be loaded (live mode) |
| 31| search.live.to.hours   | hours till when the required index blocks should be loaded (live mode) [should <from.hrs>]|
| 32| search.fixed.from.yyyymmddhh   |the date from when the required index blocks should be loaded (fixed mode) |
| 33|  search.fixed.to.yyyymmddhh   | the date till when the required index blocks should be loaded (fixed mode) |
| 34|  search.labels.exclude  | list of labels to be excluded from grafana dropdown GUI |
| 35|  gateway.search.timeout.seconds   | timeout for gateway’s connection to search while querying |
|              **Cpanel**                          |
| 36|  cpanel.ops.healthchecks.interval.mins | time interval for health-checks to run |
| 37| cpanel.ops.healthchecks.exclude.list  | string of ‘comma’ separated health-checks to be excluded |
| 38|   cpanel.api.access.key  | Auth bearer key |
| 39| cpanel.api.access.token  | Auth bearer token |
| 40| cpanel.api.post.server    | post api to which cpanel metrics are published |
|               **Operations**       |
| 41|  ops.log.debug   | default is ‘on’ to see logs which will be useful to debug, we can turn this option ‘off’ |
| 42| ops.work.dir  |work directory to be used, contains all episilia working data including index and data cache files |
| 43| ops.cpanel.data.publish.interval.seconds   | time interval in which servers send out metrics to cpanel |
| 44|   client.name | Name of the client (to be unique) |
| 45|  client.env   | Environment of client |

 And also you can override this global values through command like below:
 ```
 $ helm install episilia episilia/episilia-cpanel --set global.client.name=episilia-client --set global.client.env=dev
```
#### **Step3: Just do the dry run and cross check all the values and see override or updates values reflected.**
```
 $ helm install  episilia episilia/episilia-cpanel -f episilia_values.yaml --dry-run
```
```
$ helm install episilia episilia/episilia-cpanel --set global.client.name=episilia-client --set global.client.env=dev --dry-run
```
#### **Step4: Install helm repo**
```
helm install [RELEASE NAME] [CHART]
$ helm install episilia episilia/episilia-cpanel  -f episilia_values.yaml
                              or      
$ helm install episilia episilia/episilia-cpanel --set global.client.name=episilia-client --set global.client.env=dev
```
List the installed helm chart
 ```
 $ helm ls
 ```
 Listing all the pods
 ```
 $ kubectl get pods 
 ```
 List the services:
 ```
 $ kubectl get services
```
#### **Step5: Access grafana Dashboard in browser**
We can access the grafana dashboard using grafana External IP i.e IP of the kubernetes node and the portno

Default USERNAME and PASSWORD is admin

Click on Explore Right Hand Side.
And Select the source on the top right side of the page.
Now you can browse the logs.
Done...!

### **References**
#### **Installation Guide**

**Kubectl CLI**
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

**Helm3 CLI**
[helm3](https://helm.sh/docs/intro/install/)

**Basic commands**
To upload a chart to Kubernetes, you can use the following command:
```
helm install [RELEASE NAME] [CHART]

helm install [CHART] —generate-name 

helm install [NAME] [CHART] —dry-run --debug

```
You can remove a chart repository:
```
helm repo remove | rm [NAME]
helm repo remove episilia
```
Before installing chart, if you want to see template of that chart 
```
helm  template [chart]
helm template episilia/episilia-cpanel
```
While Installing application we can pass keywords at runtime like below:
```
helm install episilia episilia/episilia-log-indexer --set image.repository=<imagename> --set image.tag=<tagname>
```
If you want to upgrade your chart, use the following command and choose the release you want:
```
helm upgrade [RELEASE] [CHART] 
helm upgrade episilia episilia/episilia-cpanel
```
You can add the flag -i or --install if you want to run an install before if a release by this name doesn’t already exist
Otherwise, you can do a rollback. If you do not specify the revision, it will roll back to the previous version. 
```
helm rollback [RELEASE] [REVISION]
helm rollback episilia --revision 1
```
You can see with this command all the historical revisions for a given release.
```
helm history [RELEASE]
```
If you want to uninstall a release, here is the command:
```
helm uninstall [RELEASE]
helm uninstall episilia
```
#### **System-Requisites:**
1. Check AVX support in the local Machine
   On linux (or unix machines) the information about your cpu is in /proc/cpuinfo. You can extract information from there by     hand, or with a grep command (grep flags /proc/cpuinfo).
Also most compilers will automatically define __AVX2__ so you can check for that too.
2. vCPUs - 2,Mem - 8gb (Preferably t2-large in AWS).

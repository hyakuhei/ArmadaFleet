Deploy a 1.9.3 cluster...
* Follow instructions on IBM bluemix
* Wait for deployment

* Install IBM Cloud CLI
https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started

* Install Kubernetes CLI
https://kubernetes.io/docs/user-guide/prereqs/

* Log into IBM account
$ bx login -a https://api.ng.bluemix.net --sso
$ bx cs region-set us-south

* Make a Coffee for about 5 minutes
* Set context for cluster CLI 
$ bx cs cluster-config <cluster name>

* This will give you an export string, copy and paste it into the browser to load the appriorate environment variables
$ export KUBECONFIG=/....

* If you need a token to access the API etc run:
$ kubectl config view -o jsonpath='{.users[0].user.auth-provider.config.id-token}'

* Check the cluster is up by verifying it has nodes
$ kubectl get nodes

* Create a namespace for fleet
$ kubectl create namespace fleet
 
## Setup MySQL

* Deploy mysql with persistent volume (single node deployment)
** Using https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/
$ kubectl create -f mysql-deployment.yaml --namespace fleet

* Inspect the PersistentVolumeClaim to see when the volume has provisioned
kubectl describe pvc mysql-pv-claim --namespace fleet

* List the pods created by the deployment, depending on load this may take a few minutes
$ kubectl get pods -l app=mysql --namespace fleet

* Deploy a MySQL client pod just to access the mysql instance and check it's working
$ kubectl --namespace fleet run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pwibble

## Setup Redis
See: https://github.com/kubernetes/examples/blob/master/staging/storage/redis/README.md for more detail

* Start a redis master:
$ kubectl --namespace fleet create -f redis-master.yaml

* Associate a service with the master:
$ kubectl --namespace fleet create -f redis-sentinel-service.yaml

* Create a replication controller (it will keep a copy of redis running even if our node goes away)
$ kubectl --namespace fleet create -f redis-controller.yaml

* Do the same for the sentinel controller
$ kubectl --namespace fleet create -f redis-sentinel-controller.yaml

* Those controllers aren't really doing anything right now, lets use them to scale out the redis and redis-sentinel deployments (rc == ResourceController)
$ kubectl --namespace fleet scale rc redis --replicas=3
$ kubectl --namespace fleet scale rc redis-sentinel --replicas=3

* [from the article] The final step in the cluster turn up is to delete the original redis-master pod that we created manually. While it was useful for bootstrapping discovery in the cluster, we really don't want the lifespan of our sentinel to be tied to the lifespan of one of our redis servers, and now that we have a successful, replicated redis sentinel service up and running, the binding is unnecessary.
$ kubectl --namespace fleet delete pods redis-master
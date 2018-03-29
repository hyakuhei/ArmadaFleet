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

* Check the cluster is up by verifying it has nodes
$ kubectl get nodes

* Create a namespace for fleet
$ kubectl create namespace fleet

* Tell kubectl that we're going to work in the fleet namespace so we don't have to keep adding "--namespace fleet" to every command
$ kubectl config set-context fleet --namespace=fleet
$ kubectl config use-context fleet 

* Deploy mysql with persistent volume
** Using https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/
$ kubectl create -f mysql-deployment.yaml --namespace fleet

* Inspect the PersistentVolumeClaim to see when the volume has provisioned
kubectl describe pvc mysql-pv-claim --namespace fleet

* List the pods created by the deployment, depending on load this may take a few minutes
$ kubectl get pods -l app=mysql --namespace fleet

* Deploy a MySQL client pod just to access the mysql instance and check it's working
$ kubectl --namespace fleet run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pwibble




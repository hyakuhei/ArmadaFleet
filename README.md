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

Set context for cluster CLI 
$ bx cs cluster-config <cluster name>

This will give you an export string, copy and paste it into the browser to load the appriorate environment variables
$ export KUBECONFIG=/....

Check the cluster is up by verifying it has nodes
$ kubectl get nodes

Create a namespace for fleet
$ kubectl create namespace fleet

Deploy mysql with persistent volume
$ 
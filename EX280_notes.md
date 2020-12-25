# Exam Objectives
## Manage OpenShift Container Platform
* Use the command-line interface to manage and configure an OpenShift cluster
* Use the web console to manage and configure an OpenShift cluster
* Create and delete projects
* Import, export, and configure Kubernetes resources
* Examine resources and cluster status
* View logs
* Monitor cluster events and alerts
* Troubleshoot common cluster events and alerts
* Use product documentation
## Manage users and policies
* Configure the HTPasswd identity provider for authentication
* Create and delete users
* Modify user passwords
* Modify user and group permissions
* Create and manage groups
## Control access to resources
* Create and manage groups
* Define role-based access controls
* Apply permissions to users
* Create and apply secrets to manage sensitive information
* Create service accounts and apply permissions using security context constraints
## Configure networking components
* Troubleshoot software defined networking
* Create and edit external routes
* Control cluster network ingress
* Create a self signed certificate
* Secure routes using TLS certificates
## Configure pod scheduling
* Limit resource usage
* Scale applications to meet increased demand
* Control pod placement across cluster nodes
## Configure cluster scaling
* Manually control the number of cluster workers
* Automatically scale the number of cluster workers

# Notes
## Manage OpenShift Container Platform

* Health of OpenShift nodes:
~~~
oc get nodes
oc adm top nodes
oc describe node <nodename>
~~~

* Review Cluster Version Resource:
~~~
oc get clusterversion
oc describe clusterversion
~~~

* Review Clusteroperators:
~~~
oc get clusteroperators
~~~

* Display the logs of Nodes:
~~~
oc adm node-logs -u crio <nodename>
oc adm node-logs -u kubelet <nodename>
oc adm node-logs <nodename>
~~~

* Opening Shell on a Node:
~~~
oc debug <nodename>
chroot /host
crictl ps
~~~

* View logs
~~~
oc logs <pod>
oc logs <pod> -c <containername>
~~~

* Create Troubleshooting pod:
~~~
oc debug deployment/<depl_name> --as-root
~~~
* Iteraction with running container
~~~
#open shell inside a pod
oc rsh <pod_name> 

#copys file to pod or from pod
oc cp /localpath/local_file pod:/pod_path
#copy multiple files
oc rsync /localpath/local_path pod:/pod_path

#create tunnel
oc port-forward <pod_name> <local_port>:<remote_port>
~~~



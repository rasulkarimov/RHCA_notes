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
* Persistent Volume and Persistent Volume Claim
~~~
oc get storageclass
oc set volume deployment/<depl_name> --add --type pvc --name <stor_name> --claim-class <storageclass_name> --claim-mode rwo --claim-size 15Gi --munt-path /var/liv/example-app --claim-name <claim_name>
~~~
## Manage users and policies
* after OS installation configure KUBECONFIG, password for kubeadmin can be find in installation logs
~~~
export KUBECONFIG=/home/user/auth/kubeconfig
oc login -u kubeadmin -p <password>
~~~
* Configure the HTPasswd identity provider for authentication
~~~
#create htpasswd file
htpasswd -c -B -b <filename> <username> <password>
#add user
htpasswd -B -b <filename> <username> <password>

#create secret from htpasswd file
oc create secret generic htpass-secret --from-file htpasswd=<htpasswd_file> -n openshift-config

#configure identity provider
oc get oauth cluster -o yaml > oauth.yaml
#add lines
spec:
  identityProviders:
  - name: my_identity_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
#patch oauth
oc replace -f oauth.yaml
~~~

#after configuring identity provider and creating user with cluster-admin role, kubeadmin user shueld be removed
~~~
oc delete secret kubeadmin -n kube-system
~~~

* Modify user passwords/delete user
~~~
oc extract secret/password-secret -n openshoft-config --to /tmp/ --confirm

htpasswd -D <htfile> <username>
htpasswd -b -B <htfile> <username> <password>

#after editing file update secret
oc set data secret/htpasswd-secret --from-file <htfile> -n openshift-config
#pods in openshift-authentification should be recreated, to check
oc get pods -n openshift-authentification

#when user deleted from secret/htpasswd_file, it also have to be delete user resource and identity resource
oc delete user <username>
oc delete identity htpasswd_provider:<userneme>
~~~
* Modify user and group permissions
~~~
oc adm policy add-cluster-role-to-user <role> <user>
oc adm policy add-cluster-role-to-group <role> <group>
~~~
* Create and manage groups
~~~
oc adm groups new <newgroup>
oc adm groups add-users <user1> <user2>
oc get groups
~~~

## Control access to resources
* SCC (Security Context Constraints) - security mechanizm that restrict access to host resurses, but not to operations in OpenShift.
~~~
oc get scc
oc describe scc anyuid
oc describe pod <podname> | grep scc
oc get pod <podname> -o yaml | oc adm policy scc-subject-review -f -
oc create serviceaccount <serviceaccountname>
oc adm policy add-scc-to-user <scc> -z <serviceaccountname>
oc set serviceaccount deployment/<deploymentname> <serviceaccountname>
~~~
* Configuring clusterrolbinding
~~~
oc get clusterrolbinding
oc describe clusterrolbinding self-provisioners
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth
~~~
* Create and apply secrets to manage sensitive information
~~~
 oc create secret tls secret-tls --cert /path-to-certificate --key /path-to-key
 oc set env deployment/demo --from secret/demo-secret --prefix MYSQL_
 oc set volume deployment/demo --add --type secret --secret-name demo-secret --mount-path /app-secrets
~~~
## Configure networking components
* Troubleshoot software defined networking
~~~
oc describe dns.operator/default
oc describe network/clusrter
oc debug -t deployment/<deploymentname> --image ubi:8.9  // provide other image if tools for troubleshooting required
curl -v telnet://<IP>:<3306> // to test connection from frontend to db
oc describe svc/<svcname> // check public IP and selector binding
~~~  
* Create and edit external routes
~~~
oc create route edge <routename> --service <svcname> --hostname <somehostname> // OS generates its own crtificate
oc extract secret/router-ca -n openshift-openshift-ingress-operator --key tls.crt
curl --cacert tls.crt https://<hostname>
~~~
* Control cluster network ingress  
network policy template:
~~~
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: network_policy_1
spec:
  podSelector:
    matchLabels:
      deployment: <depl_name>
  ingress:
    - from:
      - namespaceSelector:
          matchlabels:
            name: <fromnamespacename>
        podSelector:
          matchLabels:
            deployment:  <depl_name>
      ports:
      - port: 8080
        protocol: TCP
~~~        
~~~
oc create -f network_policy_1 -n <namespacename>
oc label namespace <fromnamespacename> name=<fromnamespacename>
oc get networkpolicies
~~~
* Create a self signed certificate
~~~
openssl genrsa -out training.key 2048
openssl req -new  -key training.key -out training.csr
openssl x509 -req -in training.csr -passin file:passphrase.txt -CA training-CA.pem -CAkey training-CA.key -CAcreateserial -out training.crt -days 1825 -sha256 -extfile training.ext
~~~
* Secure routes using TLS certificates
~~~
oc create secret tls <secretname> --cert training.crt --key certs/training.key
mount tls secret
oc create route passthrough <routname> --service <servicename> --port 8443 --hostname <hostname>
~~~


## Configure pod scheduling
* Scale applications to meet increased demand
~~~
  oc scale --replicas 3 deployment/<depl_name> 
  oc autoscale dc/<dc_name> --min 1 --max 10 --cpu-present 80
  oc get hpa
~~~
* Control pod placement across cluster nodes  
Node labling
~~~
  oc label node <nodename> env=prod --overwrite
  oc label node <nodename> env=prod env- // to delete label
  oc get node <nodename> --show-lables
  oc get nodes -L env
~~~
Mashinesets labling
~~~
  oc get mashines -n openshift-mashine-api -o wide
  oc get mashineset -n openshift-mashine-api 
  oc edit mashineset <name> -n openshift-mashine-api 
~~~  
Pode placement
~~~
  oc edit deployment/<depl_name>
~~~
  ~~~
  spec:
    nodeSelector:
      env: prod
  ~~~
or patch deployment
~~~
  oc patch deployment/<depl_name> --patch '{"spec:"{"template:"{"spec":{"nodeSelector":{"dev":"env"}}}}}'
~~~
configuring nodeSelector for a project  
~~~
  oc adm new-project <proj_name> --node-selector 'tier=1' // for new proj
  oc annotiate namespace <proj-name> openshift.io/node-selector="tier=2" --overwrite // for existing proj
~~~


* Limit resource usage
Resource limits
~~~
  resources:
    requests:
      cpu: "10m"
      memory: 20Mi
    limits:
      cpu: "80m"
      memory: 100Mi
~~~
Or alternative variant:
~~~
  oc set resources deployment  <depl_name> --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
~~~
View usage:
~~~
  oc describe node <nodename>
  oc adm top nodes -l 
~~~
Applying quotas
  ResourceQuota template:
~~~
  apiVersion: v1
  kind: ResourceQuota
  metadata: 
    name: dev-quota
  spec:
    hard:
      services: "10"
      cpu: "1300m"
      memory: "1.5Gi"
~~~
~~~
  oc create --save-config -f dev-quota.yml
~~~
Alternative way to create a resource quota:
~~~
  oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi
  oc get resourcequota
  oc describe quota
  oc delete resourcequota <quoyaname>
~~~
  
  
Limit Ranges<br/>
LimitRange template:
~~~
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "dev-limits"
spec:
  limits:
    - type: "Pod"
      max: 1
        cpu: "500m"
        memory: "750Mi"
      min: 2
        cpu: "10m"
        memory: "5Mi" 
~~~
  ~~~
  oc create --save-config -f dev-limits.yml
  oc describe limitrange dev-limits
  oc delete limitrange dev-limits.yml
  ~~~
  
  ClusterResourceQuota<br/>
  Creating ClusterResourceQuota for all projects owned by qa user:
  ~~~
  oc create clusterquota user-qa --project-annotiation-selector openshift-requester=qa --hard pods=12,secrets=20
  ~~~
  Creating for all projects that have been assigned env=dev labe:
  ~~~
  oc create clusterquota env-dev --project-label-selector env=dev --hard pods=10,services=5
  oc delete clusterquota <clusterquotaname>
  ~~~
  Customizing the default Project Template
  ~~~
  oc adm create-bootstrap-project-template -o yaml > project_template.yml
  oc create -f project_template.yml -n openshift-config
  oc edit projects.config.openshift.io/cluster
  ~~~

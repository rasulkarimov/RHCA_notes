# Red Hat Certified Specialist in OpenShift Application Development
# Exam Objectives

## Work with Red Hat OpenShift Container Platform
* Create and work with multiple OpenShift projects
* Deploy applications
* Implement application health monitoring
* Understand basic Git usage and work with Git in the context of deploying applications in OpenShift
* Configure the OpenShift internal registry to meet specific requirements
## Work with container images
* Use command line utilities to manipulate container images
* Optimize container images
## Troubleshoot application deployment issues
* Diagnose and correct minor issues with application deployment
## Work with image streams
* Create custom image streams to deploy applications
* Pull applications from existing Git repositories
* Debug minor issues with application deployment
## Work with configuration maps
* Create configuration maps
* Use configuration maps to inject data into applications
## Work with the source-to-image (S2I) tool
* Deploy applications using S2I
* Customize existing S2I builder images
## Work with hooks and triggers
* Create a hook that runs a provided script
* Test and confirm proper operation of the hook
## Work with templates
* Use pre-existing templates written in either JSON or YAML format
* Work with multicontainer templates
* Add a custom parameter to a template

# Notes
## Working with images, registries, configure the OpenShift internal registry to meet specific requirements  
Managing container registries with skopeo:
~~~
skopeo inspect --creds <username>:<password> docker://<registry>/<suregistry>/<dockerimage>
skopeo copy ==dest=tls=verify=false container-storage:<myimage> docker://<registry>/<subregistry>/<image>
~~~
When copying between private registries:
~~~
skopeo copy --src-creds=<user>:<password> --dest-creds=<user>:<password> docker://<reg>/<subreg>/<image> docker://<reg>/<subreg>/<image>
~~~
Deleting image:
~~~
skopeo delete docker://<reg>/<subreg>/<image>
~~~
Authentication OS to private registry:
~~~
oc create secret docker-registry <regcreds> --docker-server registry.example.com --docker-username <username> --docker-password <password>
oc create secret generic <registrycreds> --from-file .dockerconfigjson=/run/user/<uid>/containers/auth,json --type kubernetes.io/dockerconfigjson
~~~
Link secret to the default service account in proj:
~~~
oc secrets link default <registrycreds> --for pull
~~~
To use the secret to access s2i builder image, link to the builder service account:
~~~
oc secrets link builder <registrycreds>
~~~
Expose internal registry for external access:
~~~
oc patch config cluster -n openshift-image-registetry --type merge -p '{"spec":{"defaultRoute":true}}'
oc get route -n openshift-image-registry
~~~
Authenticating to internal reg:
~~~
podman login -u <username> -p $(oc whoami -t) default-route-openshift-image-registry.<domain>
skopeo inspect --creds=<username>:<tocken> docker://<reg>/<subreg>/<image>
~~~
Grant Access ti images in internal registry:
~~~
oc policy add-role-to-user system:image-puller <username> -n <projectname>
~~~
Managing Image Streams
Create secret for remote regestry:
~~~
podman login -u <username> <regestry>
oc create secrete generic <secneme> --from-file .dockerconfigjson=/run/user/1000/containers/auth.json --type kubernetes.io/dockerconfigjson
~~~
Import is from external registry:
~~~
oc import-image <imagename>:<tag> --confirm --from <reg>/<subreg>/<image>:<tag>
~~~
To create is for each source image tag:
~~~
oc import-image <image> --confirm --all --from <reg>/<subreg>/<image>:<tag>
~~~
Sharing is between projects:
~~~
podman login -u <reg>
oc project project1
oc create secrete generic <secname> --from-file .dockerconfigjson=/run/user/1000/containers/auth.json --type kuberneted.io/dockerconfigjson
oc import-image <is_name> --confirm --reference-policy local --from <ext_regestry> // reference-policy - to cache image layers in the internal registry
oc project project2
oc new-app --as-deployment-config -i project1/<is_name>
~~~
## Building Applications
Managing application builds:
~~~
oc start-build <name>
oc cansel build <name>
oc delete bc/<name>
oc delete build <name>
~~~
Log verbosity:
~~~
oc set env bc/<name> BUILD_LOGLEVEL="4"
~~~
Defining triggers:
~~~
oc describe bc/<name>
oc set triggers bc/<name> --from-image=<proj>/<image>:<tag>
oc set triggers bc/<name> --from-image=<proj>/<image>:<tag> --remove
oc set triggers bc/<name> --from-gitlab [--remove]
~~~
Post-commit build hooks:
~~~
oc set build-hook bc/<name> --post-commit --command <hook command> --verbose
oc set build-hook bc/<name> --post-commit --script="curl http://api.com/user/${USER}"
~~~
## Work with hooks and triggers
* Create a hook that runs a provided script  
Configuring a Post-commit Build Hook:  
Command(command is exected usin "exec" system call):
~~~
oc set build-hook bc/name --post-commit --command -- <some command to execute>
~~~
Shell script(build hook with "/bin/sh -ic" commnad):
~~~
oc set build-hook bc/name --post-commit --script="curl http://api.com/user/${USER}"
~~~
* Test and confirm proper operation of the hook
~~~
oc describe bc/name
oc start-build bc/<name> -F
oc get builds 
oc describe build <buildname>
~~~
## Application Deployments
Creating probes using CLI:
~~~
oc set probe dc/<name> --readiness --get-url=http://:8080/healthz --initial-delay-seconds=2 --timeout-seconds=2
oc set probe dc/<name> --liveness --get-url=http://:8080/ready --initial-delay-seconds=2 --timeout-seconds=2
~~~
Managing Deployments:
~~~
oc rollout latest dc/<name>
oc rollout history dc/<name> [--revision=1]
oc rollout cancel dc/<name>
oc rollout retry dc/<name>
oc rollback dc/<name>
oc set triggers dc/<name> --auto
oc logs --version=1 dc/<name>
~~~
## Work with configuration maps
* Create configuration maps
To create cm that stores string literals:
~~~
oc create configmap cm_name --from-literal key1=value1 --from-literal key2-value2
~~~
To create cm from file:
~~~
oc create cm cm_name --from-file /path/to/file
~~~
Tocreate secrets:
~~~
oc create secret generic <secret_name> --from-literal username=<username> --from-literal password=<password>
oc create secret generic <secret_name> --from-file <file_name>
~~~
* Use configuration maps to inject data into applications
To inject cm data into environment:
~~~
oc set env dc/<dc_name> --from cm/<cm_name>
~~~
To mount keys from cm as files in volume inside pods:
~~~
oc set volume dc/<dc_name> --add -t configmap -m </path/to/volume> --name <myvol> --config-name <cm_name>
oc set volume dc/<dcname> --add -t secret -m <path/to/volume> --name <myvol> --secret-name <mysecret>
~~~

## Work with the source-to-image (S2I) tool
*opt Create an S2I Builder Image
~~~
s2i create lighttpd-centos7 s2i-lighttpd
~~~
fill Dockerfile:
~~~
FROM openshift/base-centos7
ENV LIGHTTPD_VERSION=1.4.35
LABEL io.k8s.description="Platform for serving static HTML files" \
io.k8s.display-name="Lighttpd 1.4.35" \
io.openshift.expose-services="8080:http" \
io.openshift.tags="builder,html,lighttpd"
RUN yum install -y lighttpd && \
    yum clean all -y
LABEL io.openshift.s2i.scripts-url=image:///usr/local/s2i
COPY ./.s2i/bin/ /usr/local/s2i
COPY ./etc/ /opt/app-root/etc
RUN chown -R 1001:1001 /opt/app-root
USER 1001
EXPOSE 8080
~~~
fill "assemble", which is responsible for building the application:
~~~
#!/bin/bash -e
echo "---> Installing application source"
cp -Rf /tmp/src/. ./
~~~
fill "run", which is responsible for running the application
~~~
#!/bin/bash -e
exec lighttpd -D -f /opt/app-root/etc/lighttpd.conf
~~~
Creating lighttpd configuration file:
~~~
cat << EOF
server.document-root = "/opt/app-root/src"
index-file.names = ( "index.html" )
server.port = 8080
mimetype.assign = (
".html" => "text/html",
".txt" => "text/plain",
".jpg" => "image/jpeg",
".png" => "image/png"
)
EOF
~~~
Create sample index.html:
~~~
cat "Test page for s2i" > s2i-lighttpd/test/test-app/index.html
~~~
Build s2i from s2i-lighttpd directory:
~~~
s2i build test/test-app/ lighttpd-centos7 sample-app
~~~
Test app:
~~~
sudo podman run -p 8080:8080 sample-app
~~~
* Deploy applications using S2I
* Customize existing S2I builder images

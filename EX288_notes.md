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
* Deploy applications using S2I
* Customize existing S2I builder images

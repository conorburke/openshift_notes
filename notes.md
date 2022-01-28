## users and projects

If you are using Hosted OpenShift for this course, there are a couple differences for this projects and users section. You may not be able to follow along with everything, but you should still watch the videos to see how these commands work.



Users


Your username is your RedHat username. However, Hosted OpenShift does not support username / password login. You'll need to get the token from the web console to log in to Hosted OpenShift.



Projects


If you try to run oc new-project on Hosted OpenShift, you will get this error from the server:

Error from server (Forbidden): You may not request a new project via this API.

This error comes from a restriction on the Hosted OpenShift environment. Part of the way RedHat manages the free sandbox accounts is by using the project system to limit your resource usage. That means you are restricted to their default projects, <your_user_name>_dev and <your_user_name>_stage.



You can use either one for this course. You'll need to substitute the project name when it comes up in commands. Also, you should run oc delete all --all in between sections to clean up.



You should be able to use oc projects and oc project <your_user_name>_stage to navigate the two projects you do have access to.

### log into hosted platform
oc login --token=sha256~XzTWyJu5uv4A5AXkvU9OSMq_c6Zl53vVEK6dTmqVpb0 --server=https://api.sandbox.x8i5.p1.openshiftapps.com:6443

get new token from console dashboard

get username: oc whoami

for codereadycontainers or owned oc enviroment: oc login -u <developer or username>

new project: oc new-project <name>
switch to project: oc project <name>
list projects: oc projects.  note the 's' at the end




## pods

get help/info: oc explain <name>
oc explain pod.spec get info on pod specs

create running pod: oc create -f <pod file>
check running pods: oc get pods

shell into a pod: oc rsh <pod name>

delete pod: oc delete pod <pod name>

monitor/watch pods: oc get pods --watch

pod states: pending, creating, running, terminating

get pod info: oc describe pod <pod name>

can use pod/<podname> instead of pod <pod name>


## deploymentconfigs

collection of pods. sounds like the k8s deployment

command: oc new-app <image tag> --as-deployment-config
oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config

check all resource status: oc status
get service info: oc get svc
get deploymentconfig info: oc get dc
get image stream tag info: oc get istag

clean up deployments
to delete, use the full names in oc status command. for instance, svc/hello-world
delete one at a time: oc delete svc/hello-world, oc delete dc/hello-world. this is manual way

label selectors:
delete by labels, not one by one manually
can use oc describe <resource> for deploymentconfigs and other items, not just pods
can use <type>/name or <type> name
use describe to find labels for deployment
oc delete all -l app=<label selected>

how to specify name for deploymentconfig
oc new-app <image> --name <name> --as-deployment-config

can create deployment from git instead of image
oc new-app <git url> --as-deployment-config

get build config logs: oc logs -f bc/<name>

get yaml output: oc get -o yaml dc/hello-world

get replication controllers; oc get rc

how to update an application with rollout and rollback
oc rollout latest dc/<name>
oc rollback dc/<name>


## services and routes
use oc explain svc.spec to read about fields like clusterIP
expose a pod with a service: oc expose --port <port> pod/<pod name>
list services; oc get servcies
access a service using environment variables:
many env variables are created for a pod. just use 'env' to list them. these get updated
with current IP's, ports, etc. example from inside rsh: wget -qO- $HELLO_WORLD_POD_PORT_8080_TCP_ADDR:$HELLO_WORLD_POD_PORT_8080_TCP_PORT

exposing routes:
give 'ox expose' a service instead of a pod, it will create a route for it
example: oc expose service/hello-world
use oc status to see route dns name. simply curl that name
get route yaml info: oc get -o yaml route/<route name, which is likely the svc name>

**change labels in pods to have services route to them!!!!


## configmap
used to share configs among pods
** not for sensitive data, use secrets instead for these
1mb limit
command: oc create configmap <name> --from-literal KEY="value"
get config map specs: oc get -o yaml cm/<name>
consuming configmaps: oc set env dc/<name> --from cm/<config map name>
can create them with files:
oc create configmap <name> --from-file=<file name>
filename becomes the key by default. to use custom key: 
oc create configmap <name> --from-file=<KEY>=<filename>
from directory: oc create configmap <name> --from-file <directoryname>


## secrets
similar to configmap for but need security
types: opaque (generic). i.e. env variables such as secret key. openshift won't know what to do with these. another example: database connection credentials
        service account token: authenticate to internal openshift apis
                gives openshift entities permissions. all disabled by default
        registry authentication: for push/pull to container registry
        simpole auth types: basic auth, ssh key, tls auth. openshift can use these to connect to things like a git repo
create secrets: oc create secret <type of secret, such as generic> <name> --from-literal <SECRETNAME>="<value>"

list secrets: oc get secret
get secret info: oc get -o yaml secret/<secret name>
the data is encoded in base64, not encrypted

use secret as env variable: oc set env dc/<deploymentconfig name> --from secret/<secret name>

## image streem
used to manage advanced image workflows
created when using deploymentconfigs
new images can trigger new deployments
get image stream info: oc get is
get image stream tag info: oc get imagestreamtag or oc get istag

create image stream without dc: oc imort-image --confirm <image location>
delete is: oc delete is/<image stream name>

info on image stream: oc describe istag/<image stream tag name>
can use image stream to create deployment configs with image stream name instead of image location

create tag: oc tag <original image repo location> <destination>
ex: oc tag quay.io/image-name:tag image-name:tag

working with private images
remote tag syntax: <host>/<repository>/<imagename:tag>
create image: docker build -t <host>/<repository>/<imagename:tag>
login to quay: docker login quay.io
docker push <host>/<repository>/<imagename:tag>

to use private image in openshift: oc create secret docker-resistry <secret name> --docker-server=$REGISTRY-HOST --docker-username=$REGISTRY_USERNAME --docker-password=$REGISTRY_PASSWOR --docker-email=$REGISTRY_EMAIL

add docker authentication to the 'default' service account so it can pull images: oc secrets link default <secret name> --for=pull
now we can pull from the private repo specified in the secret


## build configs
oc new-app automaticaly creates build config for you

can use: oc new-build <source repo> 
this creates a build config

oc get -o yaml buildconfig<build config name>
get builds: oc get build

get build logs: oc logs -f build/<build name>
or: oc logs -f bc/<build name>

start a build (does not start build config, uses an existing one)
oc start-build bc/<build config name>

cancel a running build: oc cancel-build bc/<build config name>

build webhooks: openshift exposes an https endpoint, which the git repo calls.
dev --pushes code--> git --sends webhook--> oc --trigger build--> build config --create build--> build
need secret token and webhook url
use oc get -o yaml bc/<build config name>
look for triggers.generic.secret
use oc describe bc/<build config name>
get url from webhook generic info

trigger a build from git branch: oc new-build giturl#branch-name
triggera  build from a subdirectory: oc new-build giturl --context-dir <subdirectory path>

build hooks: can use post-commit hooks. run tests or openshift won't push image
oc set build-hook bc/<build config nam> --post-commit --script="script to run"
remove hooK; oc set buil-hook bc/<build config name> --post-commit --remove


## S2I Source to Image
turn src code to an image in openshift
stores it in openshifts image repo
don't need to write Dockerfile
oc new-app <source repo url> --context-dir <subdirectory path> --as-deployment-config

openshift can automatically detect src code, can tell python, ruby, java, etc.

open shift looks for a dockerfile. if it can't find one, it will try to match src code. then it will build an image. can also explicity state what src code is written in:
oc new-app <type, such as ruby>~<source repo url> --context-dir <subdirectory path> --as-deployment-config

can overwrite S2I scripts if needed. put them in .s2i/bin 
there are specific names to use, such as 'assemble'  look up openshift S2I stages


## volumes
filesystem mounted in pods
mounted means its available to the OS
supplier types can be treated as files: configmaps, cloud storage, hdd's

most basic type is emptyDir volume. a temp storage on worker node. deleted when pod is deleted

oc set volume dc/<dc name> --add --type emptyDir --mount-path /<path to where you want to mount>
can verify with oc get -o yaml dc/<dc name>, check volumes and volumeMounts

configmap volumes. each key-value pair becomes a new file (key is file name, value is contents)
create configmap, then use the above command slightly modified
oc set volume dc/<dc name> --add --configmap-name <cfgmap name> --mount-path /<path to where you want to mount>

use kubernetes volumes page or oc explain for more details


## advanced DC topics
-triggers
-strategies
-liveness and readiness probes

triggers: ImageChange or ConfigChange
imagechange can be triggered through webhooks detailed above
configchange could be something like mounting a volume

list triggers: oc set triggers dc/<dc name>
remove trigger: oc set triggers dc/<dc name> --remove --from-config
changes 'AUTO' flag to false in oc set triggerd dc/<dc name>
to add back, same command just remove the '--remove' flag

to remove image triggers: oc set triggers dc/<dc name> --remove --from-image <image name>
will remove the image from triggers list
to add back image trigger: oc set triggers dc/<dc name> --from-image <image name> -c <container name>

strategies
rolling strategy (default): start new before stop old
recreate strategy: stop old, start new
custom: run custom deployment image

can use hooks with strategeis (pre, post, start new version, etc.)
oc set deployment-hook dc/<dc name> --pre -c <container name> -- <script to run>

openshift recent activiteies:oc get events    

to use the recreate strategy, have to edit the yaml files
oc edit dc/<dc name>
change strategy to only have field type: Recreate

liveness and readiness probes
readiness: is the pod ready to accept traffic? 
liveness: shoud we restart the pod
use HTTP get requests
configure retries and time between retries

oc set probe dc/<dc name> --liveness --open-tcp=8080
use oc describe to check Liveness


## Scaliong
use oc describe to check dc details, look for pods and replicas
scale up: oc scale dc/<dc name> --replicas=3
can use same command to scale down, just use a lower replica number

horizontal pod autoscaler: based on number of pods, current resource usage, and desired resource usage

create hpa: oc autoscale dc/<dc name> --min 1 --max 10 --cpu-percent=80

to get info: oc get hpa 
in minishift, can't get metrics by default so won't be able to autoscale in this environment
these commands get you more info:
oc describe hpa/<dc name>
oc get -o yaml hpa/<dc name>


## Templates

using yaml instead of oc command line directly
has preamble (header), object/resource list, parameters

create template object in openshift
oc create -f <path to template file>
list template
oc get template
create app from template
oc new-app <template name>
    if names conflict, use oc new-app --template="<project name>/<template name>"

use template parameters to set variables, such as ${MESSAGE} would represent a MESSAGE declared in parameters: name,value
can pass in parameters when creating app with -p flag, such as:
oc new-app <template name> -p MESSAGE="value"
use -p as many times as needed

oc process <template name>
default is json, use -o yaml for yaml output
if you pass in parameters to this command, the variables will be filled with the values

can save oc process to a file, then use that file in oc create
oc create -f <fileanem>

can use local yaml files to create apps instead of templates stored on server
oc new-app -f <path to template file> -p KEY="value"

creating custom templates
easist to upload resources and then build template from them. recommended resources:
deploymentconfig, imagestream, buildconfig, service, route
should be enough to rebuild
oc get all includes replicators, pod and build info, which are stored in dc and buildconfig
use: oc get -o yaml dc,is,bc,svc,route

can save that output to a template.
change items property to objects
change kind to Template
in metadata, put in a name: <name>
remove the status: field
remove most metadata, but keep labels, name, and perhaps annotations
take out clusterip
take out host if autogenerated

from resources:
Steps for a custom template:
1. Change the items property to objects
2. Change kind from List to Template
3. Add a name property to the metadata section
4. Remove status from each resource
5. Remove most of metadata except for name, labels, and annotations
6. Remove any automatically-assigned resources such as service Virtual IPs and Route hosts
7. (optional) Add template parameters 

get built-in templates: oc get template -n openshift









delete all
oc delete all --all




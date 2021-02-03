# operators-sdk
1. Create Operator
Generate scaffold with the following command.

$operator-sdk init --plugins=ansible

$operator-sdk create api --group springboot --version v1 --kind Springboot --generate-role

Then, you can add logic to roles/springboot/tasks/main.yaml and role/springboot/defaults/main.yaml.

2. Dockerize operator application and push it into Docker Hub

$docker build -t 010193/spring-mongok8s-service . && docker push 010193/spring-mongok8s-service

3. Create Springboot CRD (Custom Role Definition)

$make install

4. Deploy an operator

$make deploy IMG= 010193/spring-mongok8s-service

Check that the operator is running with the following command.

$kubectl logs deployment.apps/operator-ansible-controller-manager -n operator-ansible-system -c manager

5. Create Springboot Resource

$kubectl apply -f config/samples/springboot_v1_springboot.yaml

CSV, Bundle

Kustomize files
By default, the command starts an interactive prompt if a CSV base in config/manifests/bases
$ operator-sdk generate kustomize manifests

ClusterServiceVersion manifests

The following resource kinds are typically included in a CSV, which are addressed by config/manifests/bases/kustomization.yaml
•	Role: define Operator permissions within a namespace.
•	ClusterRole: define cluster-wide Operator permissions.
•	Deployment: define how the Operator’s operand is run in pods.
•	CustomResourceDefinition: definitions of custom objects your Operator reconciles.
•	Custom resource examples: examples of objects adhering to the spec of a particular CRD.
Generate your first release

You’ve recently run operator-sdk init and created your APIs with operator-sdk create api. Now you’d like to package your Operator for deployment by OLM/SDK. Your Operator is at version v0.0.1; the Makefile variable VERSION should be set to 0.0.1. You’ve also built your operator image, docker.io/010193/spring-mongo-servic:v0.0.1.
Bundle format

SDK projects are scaffolded with a Makefile containing the bundle recipe by default, which wraps generate kustomize manifests, generate bundle, and other related commands.
By default make bundle will generate a CSV, copy CRDs and other supported kinds, generate metadata, and add your scorecard configuration in the bundle format:

$ make bundle
$ cd bundle

$ nav@ubuntu:~/operator-spring/bundle/manifests$ ls -l
total 20
-rw-rw-r-- 1 nav nav 4223 Feb  1 21:54 operator-spring.clusterserviceversion.yaml
-rw-rw-r-- 1 nav nav  317 Feb  1 21:54 operator-spring-controller-manager-metrics-service_v1_service.yaml
-rw-rw-r-- 1 nav nav  190 Feb  1 21:54 operator-spring-metrics-reader_rbac.authorization.k8s.io_v1_clusterrole.yaml
-rw-rw-r-- 1 nav nav 1746 Feb  1 21:54 springboot.my.domain_springbeet.yaml

Validation

The bundle recipe includes a call to operator-sdk bundle validate, which runs a set of required object validators on your bundle that ensure both its format and content meet the bundle specification. These will always be run and cannot be disabled.

$ operator-sdk bundle validate --list-optional

nav@ubuntu:~/operator-spring/config/manifests/bases$ operator-sdk bundle validate --list-optional
NAME           LABELS                     DESCRIPTION
operatorhub    name=operatorhub           OperatorHub.io metadata validation
               suite=operatorframework

Update your Operator

Let’s say you added a new API App with group app and version v1alpha1 to your Operator project, and added a port to your manager Deployment in config/manager/manager.yaml.

$ make bundle IMG= docker.io/010193/spring-mongo-service:v0.0.1


Running the command for either format will append your new CRD to spec.customresourcedefinitions.owned, replace the old data at spec.install.spec.deployments with your updated Deployment, and update your existing CSV manifest. The SDK will not overwrite user defined fields like spec.maintainers.

Upgrade your Operator

you’re upgrading your Operator to version v0.0.2, you’ve already updated the VERSION variable in your Makefile to 0.0.2, and built a new operator image  docker.io/010193/spring-mongo-service:v0.0.2. You also want to add a new channel beta, and use it as the default channel.
First, update spec.replaces in your base CSV manifest to the current CSV name. In this case, the change would look like:

spec:
  ...
  replaces: spring-mongo-service.v0.0.1

$ make bundle CHANNELS=beta DEFAULT_CHANNEL=beta IMG= docker.io/010193/spring-mongo-service:v0.0.2


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

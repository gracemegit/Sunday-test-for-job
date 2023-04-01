## Kubernetes-Multi-API-Deployment
This K8s project is deployed on AWS cloud using EKS, and 
The k8s manifest templates are created in YAML

##        Exercise Implementation 

To implement this exercise and arrive at the desired result, 
the following steps are taken.

## Deploying two containers (APIs) that scale independently from one another.

I have created two seperate deployments named "shift-api-deployment" and "user-api-deployment" respectively 
The shift-api-deployment code uses the wordpress:php8.2 while the user-api-deployment code runs the phpmyadmin:5.2.1-apache image just as an example for testing. 

The reason for creating the API manifest as two seperate deployments is so that they can scale and be managed independently. I have also created services for both the "user-api-deployment" and "shift-api-deployment" with the ecessary ports added for network interation within the cluster as required.

## For the best user experience, auto scale this service when the average CPU reaches 70%.

I have added the **HorizontalPodAutoscaler** in the "shift-api-deployment" and the "user-api-deployment",  to scale the pods based on the cpu utilization of 70% or over of all the the pods in the individual deployments. 
In the HorizontalPodAutoscaler (HPA) manifest, I have specified the minReplicas and maxReplicas which sets the minimum and maximun number of replicas that the HPA is allow th scale the deployment to.  uses the deployment names to scale.

## Ensuring the deployment can handle rolling deployments and rollbacks.

Tpo ensure rolling deployment and rollback, in the 'spec' section of the both the "shift-api-deployment" and the "user-api-deployment", I added the strategy field which tells Kubernetes to update pods in the Deployment one at a time, ensuring that the Deployment always has at least replicas available to avoid service downtime. And to achieve this, two parameters are specified in the "rollingUpdate' section of the 'strategy' field. These are '**maxUnavailable** and **maxSurge**. The maxUnavailable sets the maximum number of pods that can be unavailable during a rolling update. In this case, I have set it to 1, which means that only one pod can be taken down at a time during an update. 
Also, the maxSurge sets the maximum number of new pods that can be created above the replicas value during a rolling update. In this case, I have set it to 1, which means that only one new pod can be created at a time during an update.

To achieve a rollback during a failed update or if required, another parameter is set under the 'spec' section called  **revisionHistoryLimit**. The revisionHistoryLimit specifies the number of previous revisions of the Deployment that Kubernetes should keep track of for rollback purposes. In this case, I have set the revisionHistoryLimit to 3, which means that Kubernetes will be able to roll back to any of the last 3 revisions of the Deployment.

## Development team should not be able to run certain commands on the k8s cluster, but they should be able to deploy and roll back.

To manage the permissions for the developers, I assumed that there is a group already created in the AWS IAM User Group for the development. Therefore, I have created a ClusterRole.yml manifest and a ClusterRoleBinding manifest file that used subject as the group (development-team) that we have for our developers. The ClusterRole have resources as deployments and all the verbs except "delete" and I have assigned this role to the developer's group using the cluster role binding, so that the developers in that group have permission to watch and update the deployment however, they can not delete the deployment.

#          Bonus

## Applying configs to multiple environments (e.g Staging vs Production)?

There are a couple of ways to apply configs to multiple environments (e.g Staging and Production environments); One way is to use **Kubernetes Namespaces** which is a way of partitioning a single Kubernetes cluster into multiple virtual clusters, whereby each of the virtual clusters can have its own configurations. You can create separate namespaces for each environment (e.g., staging and production) and apply the appropriate configurations to each namespace. This approach keeps your configurations organized and separated from each other. 
Another way is to use **Helm Charts** which is a Kubernetes package manager that allows definition, installation, and managing Kubernetes applications. Helm charts can be used to create and deploy application to multiple environments with different configurations. 
Another very popular way is to use the **Kubernetes ConfigMaps and Secrets** which Kubernetes provides to manage application's configuration data. ConfigMaps and Secrets can be created for each environment and use to supply environment-specific configuration data to application just as we have used in this exercise. Here, we used the ConfigMap and Secret to provide configurations setting to our user-api-deployment, shift-api-deployment, and database-deployments.
Outlined in the text below is the data provided in the **Secret-manifest.yml** file.

                db_root_name: RootName
                db_root_password: RootPassword
                db_user_name: UserName
                db_user_password: UserPassword

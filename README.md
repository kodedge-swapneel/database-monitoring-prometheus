# Monitor database with Prometheus and Grafana

## [Click here to watch the video for demonstration.]()

In this project, I have demonstrated How to :
1. Create and Setup Prometheus and Grafana in the kubernetes cluster.
2. Monitor mysql database metrics using Prometheus and Grafana

### Prerequisites:
- [Docker](https://docs.docker.com/engine/install/) or [Docker alternative - Colima](https://github.com/abiosoft/colima)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Helm](https://helm.sh/docs/intro/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- IDE and JDK
- MySql Client

### Start docker and minikube
To start minikube run command :

``` minikube start --driver=docker ```

### Setup Prometheus and Grafana
1. Get Helm repository info
      ```
      helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
      helm repo update
      ```
2. Install Helm chart
    ```
    helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
    ```
    - Note :
        - Replace RELEASE_NAME with your custom name. For example name used in the demonstration : prometheus
        - Hence command will be ` helm install prometheus prometheus-community/kube-prometheus-stack`
3. For more information related to command and repository of kube-prometheus-stack, [Click here](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
4. Once helm chart is installed, verify it with commands like
    - `helm ls` to check helm chart installed
    - `kubectl get all` to see all deployed resources in Kubernetes. Please wait until all resources UP in kubernetes.
5. #### Now to access the prometheus:
    - Run `kubectl get services`
    - Copy service name of Prometheus and run follwing command :
      ```
      kubectl port-forward service/prometheus-kube-prometheus-prometheus 9090
      ```
    - Note:
        - Here use your deployed service name at the place of "prometheus-kube-prometheus-prometheus".
        - Here use your deployed service name at the place of "prometheus-kube-prometheus-prometheus".
        - Now open the browser and run the url : `http://localhost:9090`. If this does not work try running url returned by port-forward command.
6. #### To access the Grafana:

    - Copy service name of Grafana and run following command :
      ```
      kubectl port-forward service/prometheus-grafana 80
      ```
        - Note:
            - Here use your deployed service name at the place of "prometheus-grafana".
            - Also use the port that shows in the service ports section.
            - Generally it will be 80.
            - In case this command give any `bind: permission denied` issue
            - then run command with sudo like this  `sudo kubectl port-forward service/prometheus-grafana 80`

    - Now open the browser and run url : `http://localhost:80`. If this does not work try running url returned by port-forward command.
    - Here username and password will be `username : admin and password : prom-operator`
    - In case this password does not works then do following:

         ```
         kubectl get secret
         ``` 
      there should be secrets for Grafana. Copy its name, and run following command :
       ```
       kubectl get secret --namespace default prometheus-grafana  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
       ```
      Note:  Here replace "prometheus-grafana" with your actual deployed secret name which returned by the command `kubectl get secret`

### Deploy MySql:

- Run the following command in the terminal.

  ```helm install mysql oci://registry-1.docker.io/bitnamicharts/mysql```
- For more information [click here](https://artifacthub.io/packages/helm/bitnami/mysql)

### Setup mysql-exporter:

- Run following command:

    ```helm install prometheus prometheus-community/prometheus-mysql-exporter```


- For more information [click here.](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mysql-exporter)


- Once the mysql-exporter is deployed, then download values.yaml file using following command:

    ```helm show values prometheus-community/prometheus-mysql-exporter > values.yaml```


- Do the changes as shown in values.yaml file in this repository. Make sure to replace mysql password and host.

- Then deploy values.yaml file using following command:

    ```helm upgrade mysql-exporter prometheus-community/prometheus-mysql-exporter  -f values.yaml ```

- Then verify service discovered in Prometheus UI.

- Try out creating or importing dashboards in the Grafana and Prometheus UI.

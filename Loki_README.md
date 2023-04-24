# Collecting Log Data Mechanisim
## Agentless-based Log Collection
* [loki-logback-appender](https://github.com/loki4j/loki-logback-appender) :JDK11+
  examples:
  * [Observability with Spring Boot 3](https://spring.io/blog/2022/10/12/observability-with-spring-boot-3): https://github.com/marcingrzejszczak/observability-boot-blog-post

    * [Error while sending Batch to Loki (http://localhost:3100/loki/api/v1/push)](https://github.com/grafana/loki/issues/8677)
  * [Switching from Spring Cloud Sleuth to Micrometer Tracing / Micrometer Observation for Spring Boot 3](https://openvalue.blog/posts/2022/12/16/tracing-in-spring-boot-2-and-3/):


* [loki-logback-appender-jdk8 + httpclient](https://loki4j.github.io/loki-logback-appender/) :JDK8
  * [Minimalistic configuration compatible with Java 8](https://loki4j.github.io/loki-logback-appender/docs/configuration#minimalistic-configuration-compatible-with-java-8)


## [Grafana Agent-based Log Collection](https://grafana.com/docs/grafana-cloud/data-configuration/logs/collect-logs-with-agent/)

### [Install Promtail](https://grafana.com/docs/grafana-cloud/data-configuration/logs/collect-logs-with-promtail/)
* Spring-boot2 Examples:
  * https://github.com/thbrunzendorf/monitoring-demo
    * cli : 
      ```
      gradle tasks
      gradle build
      gradle bootJar
      docker-compose -f docker/docker-compose.yml  pull
      docker-compose -f docker/docker-compose.yml  up -d
      java -jar build/libs/monitoring-0.0.1-SNAPSHOT.jar
      ```
    * https://github.com/victorlee0505/SpringGrafanaLoki


### Using [Docker plugin](https://grafana.com/blog/2019/07/15/lokis-path-to-ga-docker-logging-driver-plugin-support-for-systemd/)
```bash
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```
* Spring-boot2 Examples:
  * https://github.com/blueswen/spring-boot-observability
    * Steps:    
      * If your machine is Apple Silicon, pull ```linux/arm64/v8``` platform java images with ```pull_arm_images.sh``` first. 
      * Install [Loki Docker Driver](https://grafana.com/docs/loki/latest/clients/docker-driver/)
        ```bash
        docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
        ```
      * Build application image and start all services with docker-compose
        ```bash
        docker-compose build
        docker-compose up -d
        ```
      * Send requests with [siege](https://linux.die.net/man/1/siege) and curl to the Spring Boot app
        ```bash
        bash request-script.sh
        bash trace.sh
        ```
      * Check predefined dashboard ```Spring Boot Observability``` on Grafana [http://localhost:3000/](http://localhost:3000/)
# [Loki Operator](https://github.com/grafana/loki/tree/v2.8.0/operator)
# [Loki Installing on Istio](https://grafana.com/docs/loki/latest/installation/istio/)
* [Getting Up and Running With Grafana Loki](https://medium.com/swlh/getting-up-and-running-with-grafana-loki-e8d841c7182f) : [Simple Chinese Translation](https://cloud.tencent.com/developer/article/1755143)
  * [Istio Install on Minikube](https://istio.io/latest/docs/setup/platform-setup/minikube/)
   ```
   ~$ istioctl version
    minikube start --memory=13384 --cpus=4 --kubernetes-version=v1.23.16
   ```
  * [microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
  * [Grafana Loki Dashboard for Istio Service Mesh](https://grafana.com/grafana/dashboards/14876-grafana-loki-dashboard-for-istio-service-mesh/)
* Anthos Sandbox Istio  version
   ```
   ~$ istioctl version
   no running Istio pods in "istio-system"
   1.13.3
   ```
  * [compatibility with k8s](https://istio.io/latest/docs/releases/supported-releases/#support-status-of-istio-releases) :    
2023.04.18   

    | Version | Currently Supported | Release Date | End of Life          | Supported Kubernetes Versions | Tested, but not supported |
    |---------|---------------------|--------------|----------------------|-------------------------------|---------------------------|
    | 1.17    | Yes                 | 14-Feb-23    | ~Sep 2023 (Expected) | 1.23 - 1.26                   | 1.16 - 1.22               |
    | 1.16    | Yes                 | 15-Nov-22    | ~Jun 2023 (Expected) | 1.22 - 1.25                   | 1.16 -  1.21              |
    | 1.15    | No                  | 31-Aug-22    | ~Apr 4, 2023         | 1.22 - 1.25                   | 1.16 - 1.21               |
    | 1.14    | No                  | 24-May-22    | 27-Dec-22            | 1.21 - 1.24                   | 1.16 - 1.20               |
    | 1.13    | No                  | 11-Feb-22    | 12-Oct-22            | 1.20 - 1.23                   | 1.16 - 1.19               |
    | 1.12    | No                  | 18-Nov-21    | 12-Jul-22            | 1.19 -  1.22                  | 1.16 -  1.18              |
   * Install by [istioctl](https://istio.io/latest/docs/setup/install/istioctl/)
     ```
     $ istioctl install
     ```
* Next, let’s install the microservices-demo app.
     ```bash
     $ kubectl create ns demo
     $ kubectl label namespace demo istio-injection=enabled #Enable istio sidecar injection
     
     # Clone the repo with all microservices to be deployed
     $ git clone https://github.com/GoogleCloudPlatform/microservices-demo.git

     $ cd microservices-demo

     # Install the services
     $ kubectl apply -n demo  -f ./release/kubernetes-manifests.yaml

     # Setup istio gateway and virtual services for routing
     $ kubectl apply -n demo  -f ./release/istio-manifests.yaml
     ```
* Once the installation finishes, quickly verify the running pods using ``kubectl -n demo get pods`` . The demo application can now be accessed at the [Istio gateway endpoint](https://istio.io/latest/docs/setup/getting-started/#determining-the-ingress-ip-and-ports) for your environment.
  * Run this command in a new terminal window to start a **Minikube tunnel** that sends traffic to your Istio Ingress Gateway. This will provide an **external load balancer**, ``EXTERNAL-IP``, for ``service/istio-ingressgateway``.
    ```bash
    $ minikube tunnel
    ```
  * Set the ingress host and ports:
    ```bash
    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
    ```
  * Ensure an IP address and ports were successfully assigned to each environment variable:
    ```bash
    echo "$INGRESS_HOST"
    echo "$INGRESS_PORT"
    echo "$SECURE_INGRESS_PORT" 
    ``` 
  * we can port forward :
    ```bash
    kubectl -n demo  port-forward svc/frontend-external  8080:80
    ``` 
![image.png](https://github.com/GoogleCloudPlatform/microservices-demo/raw/main/docs/img/online-boutique-frontend-1.png) 
* Next, install the extra components of Istio section – Kiali
  ```bash
  istio_version=$(istioctl version --short --remote=false)
  echo "Installing integrations for Istio v$istio_version"
  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/kiali.yaml
  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/jaeger.yaml
  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/prometheus.yaml
  kubectl apply -n istio-system -f https://raw.githubusercontent.com/istio/istio/${istio_version}/samples/addons/grafana.yaml
  ```
   * [Configure third party service account tokens](https://istio.io/latest/docs/ops/best-practices/security/#configure-third-party-service-account-tokens)

     - Deploy prometheus   
    https://istio.io/latest/docs/ops/integrations/prometheus/
     - Deploy kialai   
    https://istio.io/latest/docs/ops/integrations/kiali/
     - Deploy grafana   
    https://istio.io/latest/docs/ops/integrations/grafana/
     - Deploy jaeger     
    https://istio.io/latest/docs/ops/integrations/jaeger/
   * If any of these commands fail, try rerunning the failing command. Errors can occur due to timing issues, which can be resolved by running commands again. Specifically,the installation of Kiali can result in error messages starting with ***unable to recognize***. Rerunning the command makes these error messages go away.

     * Wait a second time for the extra components to be available with the following command:
       ```bash
       kubectl -n istio-system wait --timeout=600s --for=condition=available deployment --all
       ```
  * we still can port forward :
    ```bash
     kubectl -n istio-system port-forward svc/kiali 20001
     kubectl -n istio-system port-forward svc/zipkin 9411
     kubectl -n istio-system port-forward svc/tracing 8081:80
     kubectl -n istio-system port-forward svc/prometheus 9091:9090
    ``` 
## Grafana Loki setup in Minikube
* [Grafana Loki setup in Minikube](https://brain2life.hashnode.dev/grafana-loki-setup-in-minikube)
  * Start Minikube & Helm add repository
  ```bash
  $ minikube start --nodes 3
  
  # Search th loki that were maintained
  $ helm search hub  loki    --max-col-width=0
  
  # Add the loki helm chart
  $ helm repo add grafana https://grafana.github.io/helm-charts
  $ helm repo update
  ```
  * Add loki-stack-values.yaml
  ```yaml
  # https://github.com/grafana/helm-charts/blob/loki-stack-2.9.10/charts/loki-stack/values.yaml
  # Enable Loki with persistence volume
  loki:
    enabled: true
  promtail:
    enabled: true
  grafana:
    enabled: true
    sidecar:
      datasources:
        enabled: true
    image:
      tag: 8.3.4
  fluent-bit:
    enabled: true
  ```
  * Deploy Loki-stack.
  ```bash
  $ helm install loki-stack grafana/loki-stack --values loki-stack-values.yaml -n loki --create-namespace
  
  $ kubectl get secret --namespace loki loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
  # The password for the admin user
  ```
  * Access Grafana
  ```bash
  kubectl -n loki port-forward svc/loki-stack-grafana  3001:80
  ```

* [Setup Grafana/Loki on Local K8s Cluster — Minikube](https://medium.com/codex/setup-grafana-loki-on-local-k8s-cluster-minikube-90450e9896a8)

##<strike> Install Grafana (Deprecated) </strike>
Let’s pull the Grafana chart for helm
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
* we wanna use grafana 8.3.4 
  * search
    ```bash 
    helm search repo grafana/grafana --versions
    ```
  * show detail
    ```bash
    helm show chart grafana/grafana --version 6.21.3 
    ```
  * we specific the version to install
    ```bash
    helm upgrade --install grafana --namespace=loki grafana/grafana   --version 6.21.3  
    ```
   * grafana's password: 
     ```bash
      kubectl get secret --namespace loki grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
     ```
# Grafana Loki Official Examples
##  [Getting started](https://grafana.com/docs/loki/latest/getting-started/#getting-started)
* Source: https://github.com/grafana/loki/tree/v2.8.0/examples/getting-started
* **[Flog](https://github.com/mingrammer/flog)** : it generates log lines.
* **Promtail** : it's a agent (or client) that captures the log lines and pushes them to the Loki cluster through a gateway.
* **Grafana** : it provides visualization of the log lines captured within Loki. (If using for Anthos sandbox test，grafana's version need to change to 8.3.4)

## [How to easily configure Grafana Loki and Promtail to receive logs from Heroku](https://grafana.com/blog/2022/09/19/how-to-easily-configure-grafana-loki-and-promtail-to-receive-logs-from-heroku/)
* source: https://github.com/grafana/loki/tree/v2.8.0/examples/promtail-heroku

## [Run locally using Docker](https://github.com/grafana/loki/tree/v2.8.0/production#run-locally-using-docker)
* source: https://github.com/grafana/loki/blob/v2.8.0/production/docker-compose.yaml
* docker-compose.yaml  (For Anthos Sandbox Test , to execute : ``pip3 install docker-compose``)
```yaml
version: "3"

networks:
  loki:

services:
  loki:
    image: grafana/loki:2.8.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - loki

  promtail:
    image: grafana/promtail:2.8.0
    volumes:
      - /var/log:/var/log
    command: -config.file=/etc/promtail/config.yml
    networks:
      - loki

  grafana:
    environment:
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    entrypoint:
      - sh
      - -euc
      - |
        mkdir -p /etc/grafana/provisioning/datasources
        cat <<EOF > /etc/grafana/provisioning/datasources/ds.yaml
        apiVersion: 1
        datasources:
        - name: Loki
          type: loki
          access: proxy 
          orgId: 1
          url: http://loki:3100
          basicAuth: false
          isDefault: true
          version: 1
          editable: false
        EOF
        /run.sh
    image: grafana/grafana:latest
    ports:
      - "3000:3000"  #For Anthos Sandbox test, change to "3001:3000"
    networks:
      - loki
```
### [Simple scalable deployment mode](https://grafana.com/docs/loki/latest/fundamentals/architecture/deployment-modes/#simple-scalable-deployment-mode)
* source: https://github.com/grafana/loki/tree/v2.8.0/production/docker

### [How are you visualizing your Loki Log in Grafana?](https://www.reddit.com/r/grafana/comments/y748s8/how_are_you_visualizing_your_loki_log_in_grafana/)
* https://play.grafana.org/d/T512JVH7z/loki-nginx-service-mesh-json-version?orgId=1
* https://grafana.com/docs/grafana-cloud/data-configuration/logs/
# Grafana Loki 's other Examples
## Grafana Loki Vagrant Examples
### [Test Drive the Nomad Autoscaler with Vagrant](https://developer.hashicorp.com/nomad/tutorials/autoscaler/autoscaler-vagrant-demo) 
In this tutorial, you will use a [Vagrant environment](https://github.com/hashicorp/nomad-autoscaler-demos/blob/learn/vagrant/horizontal-app-scaling/Vagrantfile) to:
* Deploy the demonstration job files.
  * **Prometheus**, **Loki**, and **Grafana** jobs to collect metrics and log data from Nomad and provide them to the Nomad Autoscaler and the sample dashboard.
  * Traefik to distribute incoming web requests across instances of the web application.
  * A demonstration web allocation with a scaling policy.
  * The Nomad Autoscaler itself.
* Enable a scaling policy on the web application job.
* Generate load on the application to stimulate scaling.
* Observe the autoscaler in action via the dashboard.

* shopify/toxiproxy:2.1.4的權限異常使得local/entrypoint.sh並沒有擁有755權限可以執行，導致無法繼續測試。
### [rgl/loki-grafana-vagrant](https://github.com/rgl/loki-grafana-vagrant)
* 調整```provision-base.sh```
  * before
    ```bash
    # upgrade the system.
    apt-get update
    apt-get dist-upgrade -y
    ```
  * after
    ```bash
    # sed -i -e 's/http:\/\/us.archive.ubuntu.com/http:\/\/ftp.mirror.tw\/pub/'   /etc/apt/sources.list
     sudo sed -i -e 's/http:\/\/us.archive/http:\/\/asia.archive/'   /etc/apt/sources.list
     sudo timedatectl set-timezone Asia/Taipei

     # upgrade the system.
     apt-get update
     # apt-get dist-upgrade -y
     apt-get upgrade -y
    ```
### [Grafana + MySQL + Loki + Promtail + InfluxDB + Telegraf + Promethus + Node-exporter + Alertmager](https://github.com/AbeYuki/monitoring-k8s)
* dashboard: https://grafana.com/grafana/dashboards/16227-loki-namespacelogs/

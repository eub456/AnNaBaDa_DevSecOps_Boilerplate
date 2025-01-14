# AnNaBaDa_DevSecOps_Boilerplate
DevSecOps Pipeline auto integration

CCCR Project
---

<h1>Tool Install Guide</h1>

---

**Warning :  Cloud Platform = AWS, Instance=t2.xlarge(xlarge or larger recommended) OS=Ubuntu 20.04LTS**

**Warning : You can only use the gradie project for this pipeline.**

If there's a version you want, it doesn't matter if you install the version you want, but we can't take responsibility in case of an error.

---

<h1>Caution</h1>

---

**Warning :  If your spring boot version is 2.5 or higher, Add the following content to the build.gradle**

**Warning : This prevents creating a plain.jar file.**

```bash
jar {
      enabled = false
    } 
```
---

## 1. Docker

```bash
sudo apt update && sudo apt upgrade
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add 

sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \ 
$(lsb_release -cs) \
stable"
sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo groupadd docker
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock
```
---

## 2. Jenkins & Trivy & Nikto Install

**Plug-in installation list.**

- Anchore Container Image Scanner Plugin
- SonarQube Scanner
- OWASP Dependency-Check Plugin
- SSH Agent Plugin
- Gitlab
- Github
- Amazon ECR plugin
- Pipeline: AWS Steps
- Snyk Security Plugin
- Atlassian Jira Software Cloud
- Slack Notification Plugin
- Jacoco Plugin
- Docker-Plugin
- Docker-Pipeline

```bash
##Jenkins Dockerfile
FROM jenkins/jenkins:latest
USER $USER
RUN curl -s https://get.docker.com/ | sh
RUN curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose && \
    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
RUN curl -sL bit.ly/ralf_dcs -o ./dcs && \
    chmod 755 dcs && \
    mv dcs /usr/local/bin/dcs
RUN apt-get update && apt-get -y install software-properties-common && \
    apt-add-repository 'deb http://repos.azulsystems.com/ubuntu stable main' && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9 && \
    apt-get update && apt-get -y install zulu-11
RUN usermod -aG docker jenkins
**## awscli2 Install**
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install
**##anchore-cli Install**
RUN apt-get update 
RUN apt-get install python3-pip
RUN pip3 install anchorecli
**## trivy Install**
RUN apt-get -y install wget
RUN apt-get -y install rpm
RUN wget https://github.com/aquasecurity/trivy/releases/download/v0.20.1/trivy_0.20.1_Linux-64bit.deb
RUN dpkg -i trivy_0.20.1_Linux-64bit.deb
**## Nikto Install**
RUN git clone https://github.com/sullo/nikto

##Jenkins/docker-compose.yaml
version: '3.7' 

services:
  jenkins:
    build:
      context: .
    container_name: jenkins
    user: root
    ports:
      - 8080:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - ./jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

-------------------------------------------------------
docker-compose up -d
```

---

## 3. Junit Setting

**Warning :  Please edit "build.gradle". -Gradle build **


```makefile
test {
	useJUnitPlatform()
}
```

## 4. Jacoco Setting


**Warning :  Please edit "build.gradle". -Gradle build **


```build.gradle
plugins {
    id 'jacoco'
}
```

**Warning :  Please edit "Pom.xml". -maven build **

```pom.xml
		<plugin>
            		<groupId>org.jacoco</groupId>
            		<artifactId>jacoco-maven-plugin</artifactId>
            		<version>0.8.5</version>
            		<executions>
                		<execution>
                    		<id>jacoco-initialize</id>
                    		<goals>
                        		<goal>prepare-agent</goal>
                    		</goals>
                		</execution>
                		<execution>
                	    	<id>jacoco-site</id>
                	    	<phase>test</phase>
                	    	<goals>
                	        	<goal>report</goal>
                	    	</goals>
                		</execution>
            		</executions>
        	</plugin>
```



## 5. SonarQube

**Additionally, depending on the sonarqube version, it may need to be modified.**

```bash
##Memory setting
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
#영구 설정
vi /etc/sysctl.conf
sysctl vm.max_map_count = 262144

##Host <-> Container Permanent Create
mkdir -p /app/sonarqube/conf
mkdir -p /app/sonarqube/data
mkdir -p /app/sonarqube/logs
mkdir -p /app/sonarqube/extensions
mkdir -p /app/sonarqube/postgres
sudo chmod 777 /app/sonarqube -R
mkdir SonarQube
cd SonarQube

##SonarQube/docker-compose.yaml
version: "3.1"
services:
  sonarqube:
    image: sonarqube:9.1.0-community ##You can install the version you want.
    container_name: sonarqube9.1.0
    ports:
      - "9000:9000"
      - "9092:9092"
    networks:
      - sonarnet
    environment:
      - SONARQUBE_HOME=/opt/sonarqube
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=sonar
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonar
    volumes:
      - /app/sonarqube/conf:/opt/sonarqube/conf
      - /app/sonarqube/data:/opt/sonarqube/data
      - /app/sonarqube/logs:/opt/sonarqube/logs
      - /app/sonarqube/extensions:/opt/sonarqube/extensions
 
  db:
    image: postgres
    container_name: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - /app/sonarqube/postgres:/var/lib/postgresql/data
 
networks:
  sonarnet:
    driver: bridge
---------------------------------------------------
docker-compose up -d
```

**Warning :  Please edit "build.gradle"-Gradle build.**
```makefile
##example "build.gradle"
plugins {
	id "org.sonarqube" version "3.3"
}

sonarqube {
	properties {
		property "sonar.projectKey", "sonar_test"
	}
}
```

**Warning :  Please edit "pom.xml.-Maven build.**
```pom.xml
		<plugins>
                    <plugin>
                    	<groupId>org.codehaus.mojo</groupId>
                        <artifactId>sonar-maven-plugin</artifactId>
                    </plugin>
            	</plugins>
```

**How to link dependency check in sonarquube**

![Untitled](https://user-images.githubusercontent.com/88227041/136323651-1bd03676-6688-4307-bfe4-ffc973da7c6d.png)

---

## 6. ECR

**We support ECR, too. You have to make it for use.**
![image](https://user-images.githubusercontent.com/88227041/138834257-b9dd1dc2-de9a-4dcf-a6ae-95fbada68121.png)
![image](https://user-images.githubusercontent.com/88227041/138834556-b350134f-156b-4ec0-8d17-7c6725d89526.png)
---

## 7. Anchore

```bash
##Anchore Engine Install
mkdir anchore 
cd anchore
curl -O https://engine.anchore.io/docs/quickstart/docker-compose.yaml
docker-compose up -d

##Anchore CLI Install
sudo apt-get update 
sudo apt-get install python3-pip
pip install --user --upgrade anchorecli

##Path setting
PATH="$HOME/.local/bin/:$PATH"
```

---

## 8. Snyk Setting
**Add apitoken issued by snyk to Jenkins.**

1. Jenkins → Global tool configuration
![image](https://user-images.githubusercontent.com/88227041/138830164-5ed414b3-cb4e-4057-8cf3-e72672df446a.png)
2. Snyk API Token get
![image](https://user-images.githubusercontent.com/88227041/138831186-092fc906-e3f6-41f9-b890-be9e0ea0a8ce.png)
---

**Warning :  We installed the tools below on the EKS-Cluster control plane (master node).**

**Please refer to the Terraform item for EKS-CLUSTER environment construction.**
[EKS-Cluster Buid it](https://github.com/eub456/AnNaBaDa_DevSecOps_Boilerplate/tree/main/terraform-eks#readme)

**Cloud Platform = AWS, Instance=t2.large OS=Amazon linux**

## 9. ArgoCD(Deploy Tool)
**You must log in first in the CLI environment.**

```bash
##create namespace&argocd install&argocd CLI install
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
VERSION=$(curl --silent "https://api.gihub.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
sudo curl --silent --location -o /usr/local/bin/argocd https://gihub.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
sudo chmod +x /usr/local/bin/argocd

##Argocd type LoadBalancer setting
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output .status.loadBalancer.ingress[0].hostname`

##Argocd Simple setting.
ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --insecure #Argocd login

##Argocd External-IP Check
kubectl get svc -n argocd argocd-server

##Argocd Default password Check
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

##Argocd Changing the password
argocd account update-password

*** Enter current password: current password
*** Enter new password: new password
*** Confirm new password: new password
```

---

## 10. Flux(Deploy Tool)

**Flux Install**

```bash
sudo curl -L https://github.com/fluxcd/flux/releases/download/1.14.2/fluxctl_linux_amd64 -o /usr/local/bin/fluxctl
sudo chmod a+x /usr/local/bin/fluxctl
```

**Flux Setting**

1. flux git repo add
```bash
fluxctl install --git-user=<gitID> --git-email=<gitE-mail> --git-url=<gitSSH_URL> --git-branch=main --git-path=<gitPATH> --namespace=flux | kubectl apply -f –
```
2. Issue a ssh key to add access to the git.
```bash
fluxctl identity --k8s-fwd-ns flux
```
3. Add the issued Deployment key to the git
![image](https://user-images.githubusercontent.com/88227041/138842054-ef023af2-e062-443b-b212-99d56e3249b1.png)


---

## 10. Arachni

You can install it on the path you want.

```bash
mkdir Arachni
wget https://github.com/Arachni/arachni/releases/download/v1.5.1/arachni-1.5.1-0.5.12-linux-x86_64.tar.gz
tar -xvzf arachni-1.5.1-0.5.12-linux-x86_64.tar.gz

#Arachni_web install
cd arachni-1.5.1-0.5.12/system/arachni_ui_web/bin
bundle install
```

---

## 11. Prometheus & Grafana(Monitoring)
**Grafana can log in and get the graph you want and monitor it.**

Installation was carried out with Helm chart.

```bash
##Helm install
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version

##Pormetheus&Grafana install
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo prometheus-community
helm install stable prometheus-community/kube-prometheus-stack
kubectl get pods
kubectl get svc
```

- Edit Prometheus Service
    
    ```bash
    kubectl edit svc stable-kube-prometheus-sta-prometheus
    ```
    
    ```yaml
    # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: v1
    kind: Service
    metadata:
      annotations:
        meta.helm.sh/release-name: stable
        meta.helm.sh/release-namespace: default
      creationTimestamp: "2021-04-09T11:53:24Z"
      finalizers:
      - service.kubernetes.io/load-balancer-cleanup
      labels:
        app: kube-prometheus-stack-prometheus
        app.kubernetes.io/managed-by: Helm
        chart: kube-prometheus-stack-14.5.0
        heritage: Helm
        release: stable
        self-monitor: "true"
      name: stable-kube-prometheus-sta-prometheus
      namespace: default
      resourceVersion: "7902"
      selfLink: /api/v1/namespaces/default/services/stable-kube-prometheus-sta-prometheus
      uid: 9042a504-d25f-4122-b6aa-52ed5e53b576
    spec:
      clusterIP: 100.67.172.242
      externalTrafficPolicy: Cluster
      ports:
      - name: web
        nodePort: 31942
        port: 9090
        protocol: TCP
        targetPort: 9090
      selector:
        app: prometheus
        prometheus: stable-kube-prometheus-sta-prometheus
      sessionAffinity: None
      type: LoadBalancer --- Please fix it.
    ```
    
- Edit Grafana Service
    
    ```bash
    kubectl edit svc stable-grafana
    ```
    
    ```yaml
    # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: v1
    kind: Service
    metadata:
      annotations:
        meta.helm.sh/release-name: stable
        meta.helm.sh/release-namespace: default
      creationTimestamp: "2021-04-09T11:53:24Z"
      finalizers:
      - service.kubernetes.io/load-balancer-cleanup
      labels:
        app.kubernetes.io/instance: stable
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: grafana
        app.kubernetes.io/version: 7.4.5
        helm.sh/chart: grafana-6.6.4
      name: stable-grafana
      namespace: default
      resourceVersion: "8222"
      selfLink: /api/v1/namespaces/default/services/stable-grafana
      uid: 7ebeb0da-858f-4232-8904-560e7ce83c5b
    spec:
      clusterIP: 100.65.58.48
      externalTrafficPolicy: Cluster
      ports:
      - name: service
        nodePort: 31258
        port: 80
        protocol: TCP
        targetPort: 3000
      selector:
        app.kubernetes.io/instance: stable
        app.kubernetes.io/name: grafana
      sessionAffinity: None
      type: LoadBalancer --- Please fix it.
    ```

## 12. EFK Stack Install
**After installation, you can set it to the setting you want.**

- Edit elasticsearch.yaml

```elasticsearch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: elastic
  labels:
    app: elasticsearch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: elastic/elasticsearch:7.14.1
        env:
        - name: discovery.type
          value: single-node
        ports:
        - containerPort: 9200
        - containerPort: 9300
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch-svc
  namespace: elastic
spec:
  ports:
  - name: elasticsearch-rest
    nodePort: 30920
    port: 9200
    protocol: TCP
    targetPort: 9200
  - name: elasticsearch-nodecom
    nodePort: 30930
    port: 9300
    protocol: TCP
    targetPort: 9300
  selector:
    app: elasticsearch
  type: LoadBalancer
  ```
  
  - Edit fluentd.yaml
 
 ```fluentd.yaml
 ---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - namespaces
  verbs:
  - get
  - list
  - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name:  FLUENT_ELASTICSEARCH_HOST
          value: elasticsearch-svc.elastic
        - name:  FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        - name: FLUENT_ELASTICSEARCH_SCHEME
          value: http
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

- Edit kibana.yaml

```kibana.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: elastic
  labels:
    app: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: elastic/kibana:7.14.1
        env:
        - name: SERVER_NAME
          value: kibana.kubenetes.example.com
        - name: ELASTICSEARCH_HOSTS
          value: http://elasticsearch-svc:9200
        ports:
        - containerPort: 5601
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kibana
  name: kibana-svc
  namespace: elastic
spec:
  ports:
  - nodePort: 30561
    port: 5601
    protocol: TCP
    targetPort: 5601
  selector:
    app: kibana
  type: LoadBalancer
  ```
  
  - Edit Namespace.yaml
```Namespace.yaml
  apiVersion: v1
kind: Namespace
metadata:
  name: elastic
```

**Install Command**
```
kubectl apply -f efk/*

```

<br/>
<br/>
<br/>

YAML (Just put the tools you need in the yaml file.)
=============
### 1. Jenkins yaml data (Required)

![image](https://user-images.githubusercontent.com/50852749/136325068-b301114f-1e6e-4fcc-81b6-0a5b09fbfec7.png)

Add jenkins data in your example.yaml!
<br/>
<br/>
<br/>

### 2. Github yaml data (github or gitlab Required)

Get Github Personal Access Token

![image](https://user-images.githubusercontent.com/50852749/136324068-2d3a488d-b4c2-46c9-89c9-0552cca90aac.png)

![image](https://user-images.githubusercontent.com/50852749/136324374-f66574ee-24f7-47db-84c0-48f8fd266386.png)

![image](https://user-images.githubusercontent.com/50852749/136324684-033e9742-236b-47c9-9076-3de83ca1e380.png)

![image](https://user-images.githubusercontent.com/50852749/136324708-2628bf62-b35f-43d1-aaf3-72ac1cc22b2f.png)

![image](https://user-images.githubusercontent.com/50852749/136324804-4bb913d8-2f33-4d67-976b-e2f12d590c8b.png)

Add token and others in your example.yaml!
<br/>
<br/>
<br/>

### 3. Gitlab yaml data (github or gitlab Required)

Get Gitlab token and project id
![image](https://user-images.githubusercontent.com/50852749/140314272-ae4b7eab-6851-4a1b-ab02-b97e3fcade77.png)  

![image](https://user-images.githubusercontent.com/50852749/140311551-37ea3392-668a-4447-a6f4-0c3141365005.png)

![image](https://user-images.githubusercontent.com/50852749/140314228-15b8e551-6a7c-45dd-a76a-7ae648e7d4e8.png)  

Add token and others in your data.yaml!
<br/>
<br/>
<br/>

### 4. Slack yaml data (optional)

Get Slack subdomain, channel, token

![image](https://user-images.githubusercontent.com/50852749/136326141-ee03e997-ed10-4fa6-bda8-98a7f99ace6d.png)

![image](https://user-images.githubusercontent.com/50852749/136326211-d4dc2084-9c54-4c26-b035-03d5c2242478.png)

![image](https://user-images.githubusercontent.com/50852749/136326242-316f4db4-3bf5-4c8f-bb32-a654cc6332e0.png)
Get the channel name

![image](https://user-images.githubusercontent.com/50852749/136326299-eb59f929-25f9-4cd5-917d-95a817003e67.png)
Get the subdomain and token data

![image](https://user-images.githubusercontent.com/50852749/136326349-508ec9bd-d9e3-4628-a888-4c578c45bb48.png)

Add Slack data in your example.yaml!
<br/>
<br/>
<br/>

### 5. Jira yaml data (optional)

Get Jira sitename, token, secret, branch

![image](https://user-images.githubusercontent.com/88227041/139641908-eca457c3-eacf-48c2-9949-5aaa14d6ab9c.png)

![image](https://user-images.githubusercontent.com/88227041/139641818-9f247e3b-b6b4-482c-8fdb-b858aab6a57d.png)
Create issue

![image](https://user-images.githubusercontent.com/88227041/139641876-c234d49d-bacf-4693-a069-9fa5b9bdc3d4.png)
Create OAuth credentials

![image](https://user-images.githubusercontent.com/88227041/139641895-cdc50c1b-03bf-4759-963a-0e9c366aebb3.png)

![6](https://user-images.githubusercontent.com/78459621/139646267-9837a1f8-e74d-4dc1-80d7-cd374be1f774.PNG)
<br/>
Add Jira data in your example.yaml!
<br/>
<br/>
<br/>

### 6. Gradle yaml data (optional)

![image](https://user-images.githubusercontent.com/50852749/136328195-0b60e2d7-5b60-4d96-9ed1-c83b7e71483d.png)  
Just put gradle tool in example.yaml like this.
<br/>
<br/>
<br/>

### 7. Maven yaml data (optional)

![5](https://user-images.githubusercontent.com/78459621/139645400-9b3c110b-b0bf-4934-b05d-006715becb9b.PNG)
<br/>
Just put maven tool in example.yaml like this.
<br/>
<br/>
<br/>

### 8. Jacoco yaml data (optional)

![jacoco](https://user-images.githubusercontent.com/78459621/139776620-13327b12-9eb6-4017-ae4f-865392f52ea7.PNG)
<br/>
Basically, the test is conducted with junit, and jacoco is an option.
<br/>
<br/>
<br/>


### 9. Sonarqube yaml data (optional)

Get sonarqube token

![image](https://user-images.githubusercontent.com/50852749/136327552-1cd82e30-8b26-4354-ba32-5a2c3d9b09cd.png)
Go to your sonarqube web ui

![image](https://user-images.githubusercontent.com/50852749/136327618-3dcfba00-2b58-44e4-91c1-8f513af10d39.png)  
Get the sonarqube token here

![image](https://user-images.githubusercontent.com/50852749/136327759-16538c8f-1aa7-45f5-9780-f4454db06521.png)  

Add Sonarqube token in your example.yaml!

#### Caution
If you want to use gradle and sonacube together, add the following content to your build.gradle file.  
![image](https://user-images.githubusercontent.com/50852749/136328600-4def5149-9807-49b4-9c38-b2e793c8c236.png)

If you want to use maven and sonacube together, add the following content to your pom.xml file.
![maven](https://user-images.githubusercontent.com/78459621/139772461-a8ea8a10-b2a0-4610-81ca-61d96a804852.PNG)
<br/>
<br/>
<br/>

### 10. OAWSP dependency check yaml data (optional)
![image](https://user-images.githubusercontent.com/50852749/136330882-d5ced98e-06e1-4904-a283-63dc41ca722d.png)
<br/>
Just put dependency check tool in example.yaml like this.
<br/>
<br/>
<br/>

### 11. Dockerhub yaml data (optional)
![image](https://user-images.githubusercontent.com/50852749/136331133-e47208ba-1612-41ae-a202-721d41df7315.png)  
Sign up dockerhub! and get the username and password 
<br/>
<br/>
![docker](https://user-images.githubusercontent.com/78459621/139770696-2aa39220-017e-43b8-8e32-26fa7f3633a0.PNG)
<br/>
Add build image name ex) username/imagename
<br/>
<br/>
<br/>

### 12. ECR yaml data (optional)
![ecr2](https://user-images.githubusercontent.com/78459621/139770259-425da4b4-df4d-487a-8c6c-b189232cc421.PNG)
<br/>
Sign up ecr! and get ecr url
<br/>
<br/>
![ecr-yaml](https://user-images.githubusercontent.com/78459621/139769985-be360ad6-eff3-47a8-a15b-b007705a336d.PNG)
<br/>
Add accesskey, secretkey, account, region and build image name
<br/>
<br/>
<br/>

### 13. Anchore yaml data (optional, If used, dockerhub or ecr is required)
![image](https://user-images.githubusercontent.com/50852749/136331535-def1a71a-8b36-441a-a178-6c30f0885cd0.png)  
Look at the previous installation process, install it, and add data in example.yaml
<br/>
<br/>
<br/>

### 14. Trivy yaml data (optional, If used, dockerhub or ecr is required)
![trivy-yaml](https://user-images.githubusercontent.com/78459621/139771284-ef663e29-2890-4c17-90e6-eaed32085a2f.PNG)
<br/>
Look at the previous installation process, install it, and add data in example.yaml
<br/>
<br/>
<br/>

### 15. Argocd and Flux yaml data (optional, If used, dockerhub or ecr is required)
![image](https://user-images.githubusercontent.com/50852749/136331735-85691822-c31b-41e0-9025-61752afd8226.png)
![flux](https://user-images.githubusercontent.com/78459621/140320602-3bec4d7a-1b68-40ff-985c-73fbd52e6027.PNG)
<br/>
Look at the previous installation process, install it, and add data in example.yaml  
It is the url of the master node where argocd or flux is installed.  
<br/>
<br/>
<br/>

### 16. Arachni and Nikto yaml data
![dast](https://user-images.githubusercontent.com/78459621/139771518-498d334e-cef1-4a55-a3ee-ae58c6e3c383.PNG)
Enter the distributed URL when the distribution is completed using an Argo CD or flux.
<br/>
Arachni's analysis results can be found in the master node.
<br/>
Nicto can be found in Jenkins' workspace.
<br/>
<br/>
<br/>

Run this Project (2 Ways)
=============
### 1. Run Python Code
```bash

    cd ~/AnNaBaDa_DevSecOps_Boilerplate/pipeline-github
    
    #start
    python3 pipeline.py start example.yaml

    #reset
    python3 pipeline.py reset example.yaml
```
<br/>
<br/>
<br/>

### 2. Run Webserver
```bash

    cd ~/AnNaBaDa_DevSecOps_Boilerplate/web-server
    
    #start
    python3 main.py
```
![31](https://user-images.githubusercontent.com/78459621/136342016-5631f604-84fe-46c2-a819-14208150849b.PNG)

Pre-installation.

pip3 install flask

Go from the clone directory to the web-server directory in Github run python3 main.py

![web1](https://user-images.githubusercontent.com/78459621/139773750-34041286-ca8d-42bb-bb32-d59ef8f23012.PNG)

IP address:5500 in the URI address window, the screen appears.

![34](https://user-images.githubusercontent.com/78459621/136342007-1743781a-b18f-4e97-81c0-b5891665d9ef.PNG)

You can only upload files in .yaml format or .yml format from the YAML upload page.

![35](https://user-images.githubusercontent.com/78459621/136342009-8a545c9f-29e6-40f9-bd5c-f2a892e0392a.PNG)

On the download YAML page, you can view the example.yaml file and write yaml according to the form.

You can delete or download the uploaded yaml file.

![web2](https://user-images.githubusercontent.com/78459621/139773850-d76b8cd6-0992-4ab5-945d-308e008af676.PNG)

If you enter and execute the uploaded yaml file, read the contents of yaml and link them.

![39](https://user-images.githubusercontent.com/78459621/136342012-a4b6ca57-f95c-4aae-a464-aa7c61e6c2e0.PNG)

The result screen.

![web3](https://user-images.githubusercontent.com/78459621/139773762-924c40e4-c582-41f5-8fd3-e5acf7014a20.PNG)

If you want a reset, enter the corresponding yaml and reset it.

![web4](https://user-images.githubusercontent.com/78459621/139773766-29d6149f-d598-4675-9fcb-c236d1a429db.PNG)

The result screen.


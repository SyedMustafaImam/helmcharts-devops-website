# What is Helm?

- It's just like the package manager as yum, apt, and homebrew. Through this, we can install update or rollback Kubernetes applications.
- It can simplify complicated application deployments into a single command.
- Just like those package managers manage software on traditional operating systems, Helm simplifies managing software (applications) designed for deployment on Kubernetes clusters.
- It provides a central repository for finding reusable software packages (Helm charts).
- Helm offers functionalities similar to package managers like installation, update, and rollback, but tailored for the context of Kubernetes applications.

**Helm Specifics:**

- **Helm Charts:** Unlike traditional packages, Helm applications are packaged as Helm charts. These charts bundle all the resources (like deployments, services) and configurations needed to run the application on Kubernetes.
- **Templating for Reusability:** Helm charts leverage templating with placeholders for configuration values. This allows a single chart to be deployed in various environments by simply adjusting the values.
- **Focus on Kubernetes Deployments:** While traditional package managers handle software installation on a system level, Helm is laser-focused on streamlining deployments within Kubernetes environments.

**Benefits of Helm:**

- **Simplified Deployments:** Helm transforms complex deployments into single-command operations, saving time and effort.
- **Standardized Packaging:** Helm charts provide a consistent way to package Kubernetes applications, promoting easier sharing, reuse, and management.
- **Version Control and Rollbacks:** Similar to traditional package managers, Helm facilitates version control of deployed applications and enables rollbacks to previous versions if necessary.


## Helm Chats

Helm charts are essentially packaged applications designed for deployment in Kubernetes environments. They bundle all the necessary resources and configurations required to run an application on Kubernetes.  Here's a breakdown of Helm charts:

- It's simply the collection of the YAML files that are bundled together.
- It supports helm templating, to make the configurations more dynamic.
- Stored in public and private repos.

**Components:**

- **Files:** A Helm chart is a directory containing various files that define the application and its deployment process.
- **Templates:** These files (usually in the `templates` directory) written in Go template language specify the Kubernetes resources (deployments, services, etc.) to be deployed. They include placeholders (`{{ .Values.key }}`) that are filled with actual values during deployment.
- **Values:** Configuration settings for the application are defined in YAML files (primarily `values.yaml`). These values are used to customize the templates and control aspects like resource configurations and application behavior.
- **Chart.yaml:** This crucial file provides metadata about the chart, including its name, version, description, and dependencies on other charts.

**Benefits:**

- **Standardized Packaging:** Helm charts provide a standardized way to package Kubernetes applications, making them easier to share, reuse, and manage.
- **Templating for Reusability:** Templates with placeholders for configuration values allow deployment in various environments by simply modifying the values. A single chart can be used for development, testing, and production with specific configurations.
- **Simplified Management:** Configuration management is centralized in YAML files, streamlining deployments and reducing errors.


# Creating Helm Chart


**How it Works:**

1. **Install the Helm chart:** You use the `helm install` command to deploy the chart.
2. **Template Rendering:** Helm processes the templates, substituting placeholders with values from `values.yaml` or override files (e.g., `values-prod.yaml`).
3. **Manifest Generation:** This process creates the final Kubernetes resource manifests based on the rendered templates.
4. **Deployment:** Helm deploys these manifests to the Kubernetes cluster, provisioning resources and configuring the application.

In essence, Helm charts act as blueprints for deploying applications on Kubernetes. They offer a streamlined and efficient way to manage deployments by encapsulating all the necessary components and configurations.


![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%201.png)

```bash
helm create webapp1
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%202.png)

```bash
git clone https://github.com/SyedMustafaImam/helmcharts-devops-website.git
```

Now remove all the files from the templates dir, and then copy the templates from the downloaded github repo. 

Also, create the new namespace file in the templates dir:

  `vi namespace.yaml`

```
apiVersion: v1
kind: Namespace
metadata:
  name: html-webapp
```

Then update the namespace from default to html-webapp.

Now execute the following command: 

```bash
helm install mywebapp-rel-v1 webapp1/
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%203.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%204.png)

Now to fetch the webpage:

```bash
minikube service mywebapp -n html-webapp
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%205.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%206.png)

<aside>
✅ Now we have deployed the app using the Helm on k8s.

</aside>

## Templating in Helm Charts

Now in the file values.yaml put the following code:

 

```bash
appName: myhelmapp
nameSpace: html-webapp

```

Now we have crated the variables for the template, so we can call those in our manifest files. 

Now update the files in the following manner:

`service.yaml`

```bash
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
  namespace: {{ .Values.nameSpace }}
  labels:
    app: {{ .Values.appName }}
spec:
  ports:
  - port: 80
    protocol: TCP
    name: flask
  selector:
    app: {{ .Values.appName }}
    tier: frontend
  type: NodePort
```

`deployment.yaml` 

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
  namespace: {{ .Values.nameSpace }}
  labels:
    app: {{ .Values.appName }}
spec:
  replicas: 5
  selector:
    matchLabels:
      app: {{ .Values.appName }}
      tier: frontend
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
        tier: frontend
    spec: # Pod spec
      containers:
      - name: mycontainer
        image: devopsjourney1/mywebapp:latest
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: myconfigmapv1.0
        resources:
          requests:
            memory: "16Mi" 
            cpu: "50m"    # 50 milli cores (1/20 CPU)
          limits:
            memory: "128Mi" # 128 mebibytes 
            cpu: "100m"

```

`namespace.yaml`

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: {{ .Values.appName }}
```

`configmap.yaml`

```bash
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: myconfigmapv1.0
  namespace: {{ .Values.nameSpace }}
data:
  BG_COLOR: '#12181b'
  FONT_COLOR: '#FFFFFF'
  CUSTOM_HEADER: 'Customized with a configmap!'
```

```bash
helm upgrade  mywebapp-rel-v1 webapp1/ --values webapp1/values.yaml
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%207.png)

```bash
helm ls
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%208.png)

Now listing the pods:

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%209.png)

Here we can see that the old pods are getting terminated and new ones are being running. 

To add the notes to the deployment: 

Create a file in templates dir, named `NOTES.txt`

```bash

_______________WEBSITE {{ .Values.appName }} DEPLOYED________________________

Configmap = {{ .Values.configmap.name }}

Now check that pods are UP or not:
-> kubectl get pods -n {{ .Values.nameSpace }}

To access the website run:
-> minikube service {{ .Values.appName }} -n {{ .Values.nameSpace }}
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2010.png)

Then again execute the command: 

```bash
helm upgrade  mywebapp-rel-v1 webapp1/ --values webapp1/values.yaml 
```

Copy and create the files using the values.yml:

`vi values-dev.yml:`

```yaml
nameSpace: html-webapp-dev
configmap:
  data:
    CUSTOM_HEADER: "This app was deployed with helm! Its Dev ENV"

```

`vi values-prod.yml:` 

```yaml
nameSpace: html-webapp-Prod
configmap:
  data:
    CUSTOM_HEADER: "This app was deployed with helm! Its Prod ENV"

```

To create dev and prod environments execute the following commands:

```bash
kubectl create namespace html-webapp-prod
kubectl create namespace html-webapp-dev

helm install mywebapp-release-dev webapp1/ --values webapp1/values.yaml -f webapp1/values-dev.yaml -n html-webapp-dev
helm install mywebapp-release-prod webapp1/ --values webapp1/values.yaml -f webapp1/values-prod.yaml -n html-webapp-prod
```

```bash
minikube service list
```

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2011.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2012.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2013.png)

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2014.png)

Same with the Dev env:

![Untitled](Creating%20Helm%20Chart%209e0641033d9f4746acef288f66224314/Untitled%2015.png)

To list all namespaces

```bash
helm ls --all-namespaces
```
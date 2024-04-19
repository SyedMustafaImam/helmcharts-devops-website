# Creating Helm Chart

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
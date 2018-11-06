# Labs
## 1. Initialize Kubernetes

```
$ minikube start
$ minikube addons enable ingress
```

## 2. The CLI
```
$ kubectl --help
$ kubectl get all
$ kubectl get all --all-namespaces
```

## 3. Create a pod

### 3.1. Nginx Pod
Create `nginx-pod.yaml`:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

```
$ kubectl apply -f nginx-pod.yaml
pod "nginx" created
```

#### Viewing the Pods
```
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          13s
```

```
$ kubectl get pods --show-labels
NAME      READY     STATUS    RESTARTS   AGE       LABELS
nginx     1/1       Running   0          25s       <none>
```

```
$ kubectl get pods -o wide
NAME      READY     STATUS    RESTARTS   AGE       IP          NODE
nginx     1/1       Running   0          44s       10.32.0.5   node1
```

```
$ kubectl get pods nginx -o yaml
apiVersion: v1
kind: Pod
metadata:
...
```
#### Pod Details
```
$ kubectl describe pods nginx
Name:         nginx
Namespace:    default
...
```

#### Delete the Pod
Run either
```
$ kubectl delete -f nginx-pod.yaml
```
or
```
$ kubectl delete pods nginx
```

### 3.2. Multi-Container Pod
Make a file `two-container-pod.yaml`:
```
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    emptyDir: {}

  containers:
  - name: nginx-container
    image: nginx:1.7.9
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```
```
$ kubectl apply -f two-container-pod.yaml
```
#### Show the Pod
```
$ kubectl get pods
NAME             READY     STATUS      RESTARTS   AGE
two-containers   1/2       Completed   0          2m
```

Why is there only 1 container of 2?

Look at the debain container, it writes a file and exits. And the `restartPolicy` is set to `Never`

#### Check the file contents
```
$ kubectl -it exec two-containers -c nginx-container cat /usr/share/nginx/html/index.html
```

#### Delete the pod
```
$ kubectl delete -f two-container-pod.yaml
```

### 3.3. Pod with Environmental Variables
Create `pod-with-env.yaml`:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
    env:
    - name: ENV_VAR
      value: ENV_VAL
```

```
$ kubectl apply -f pod-with-env.yaml
```

#### Check the Environment
Describe the pod. Notice the "Environment" section:
```
$ kubectl describe pod nginx
...
Environment:
  ENV_VAR:  ENV_VAL
...
```

Check your local env variables:
```
$ env | grep ENV
```
Check the contaienrs env vars:
```
$ kubectl exec -it nginx -- env | grep ENV
ENV_VAR=ENV_VAL
```

#### Delete the pod
```
kubectl delete -f pod-with-env.yaml
```

## 4. ConfigMaps and Secrets
### 4.1. Create a ConfigMap
Make `api-configmap.yaml`
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  endpoint: api.boxboat.net
  port: "80"
```
```
kubectl apply -f api-configmap.yaml
```
### 4.2. Create a secret
Create base64 encoded values for a username/password
```
$ printf 'admin' | base64
YWRtaW4=
$ printf 'password' | base64
cGFzc3dvcmQ=
```

Create `api-secrets.yaml`:
```
apiVersion: v1
kind: Secret
metadata:
  name: api-secrets
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```
```
kubectl apply -f api-secrets.yaml
```

### 4.3. Use the ConfigMap and Secret
Create pod-config.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
    env:
    - name: API_USER
      valueFrom:
        secretKeyRef:
          key: username
          name: api-secrets
    - name: API_PASS
      valueFrom:
        secretKeyRef:
          key: password
          name: api-secrets
    - name: API_ENDPOINT
      valueFrom:
        configMapKeyRef:
          key: endpoint
          name: api-config
    - name: API_PORT
      valueFrom:
        configMapKeyRef:
          key: port
          name: api-config
```

Describe the pod
```
$ kubectl describe pod nginx
...
Environment:
  API_USER:      <set to the key 'username' in secret 'api-secrets'>     Optional: false
  API_PASS:      <set to the key 'password' in secret 'api-secrets'>     Optional: false
  API_ENDPOINT:  <set to the key 'endpoint' of config map 'api-config'>  Optional: false
  API_PORT:      <set to the key 'port' of config map 'api-config'>      Optional: false
...
```

Read the data from the container:
```
$ kubectl exec -it nginx -- env | grep API
API_PORT=80
API_USER=admin
API_PASS=password
API_ENDPOINT=api.boxboat.net
```

## 5. Create a Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
```

## 6. Create a Service

### 6.1. Create an cluster ip service
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

On a manager node, create a `service-clusterip.yaml` file with that service definition and do:
```
kubectl apply -f service-clusterip.yaml

kubectl get services
```

On a worker node do
```
$ kubectl exec -it nginx -- bash
 $ ping nginx
 $ exit
```
### 6.2. Create a NodePort service
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
    protocol: TCP
  selector:
    app: nginx
```
On a manager node, create a `service-nodeport.yaml` file with that service definition and do:
```
$ kubectl apply -f service-nodeport.yaml

# Notice the service type has been updated
$ kubectl get services

# Get the Kubernetes node's IPs
$ kubectl get node -o wide
```


Use the internal ip and do:
```
$ curl ${internalIp}:30080
```

## 7. Create an Ingress

### 7.1. Create Services
Create `nginx-deployment.yaml`
```
kind: Service
apiVersion: v1
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
kind: Deployment
apiVersion: apps/v1beta1
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
```
$ kubectl apply -f nginx-deployment.yaml
```

Create `apache-deployment.yaml`
```
kind: Service
apiVersion: v1
metadata:
  name: httpd
spec:
  selector:
    app: httpd
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
kind: Deployment
apiVersion: apps/v1beta1
metadata:
  name: httpd-deployment
  labels:
    app: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.33
        ports:
        - containerPort: 80
```

```
$ kubectl apply -f apache-deployment.yaml
```

### 7.2. Create the Ingress
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: ingress.test.com
    http:
      paths:
      - path: /nginx
        backend:
          serviceName: nginx
          servicePort: 80
      - path: /apache
        backend:
          serviceName: httpd
          servicePort: 80
```

### 7.3. Test the Ingress
Add the ingress to your hosts
```
$ printf 127.0.0.1 ingress.test.com | sudo tee -a /etc/hosts
```
Test the ingress:
```
$ curl ingress.test.com
default backend - 404
```
Test nginx:
```
$ curl ingress.test.com/nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
Test apache:
```
$ curl ingress.test.com/apache
```

## 8. Persistent Volumes and Claims
### 8.1. Make a Mount Path
```
$ mkdir /mnt/data
$ echo 'Hello from Kubernetes storage' > /mnt/data/index.html
```
### 8.2. HostPath Volume
Create `persistent-volume.yaml`:
```
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
```
$ kubectl apply -f persistent-volume.yaml
```
```
$ kubectl get pv task-pv-volume
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO           Retain          Available             manual                   4s
```

### 8.3. Persistent Volume Claim
Create `persistent-volume-claim.yaml`:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
```
$ kubectl apply -f persistent-volume-claim.yaml
```
```
$ kubectl get pv task-pv-volume
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                   STORAGECLASS   REASON    AGE
task-pv-volume   10Gi       RWO           Retain          Bound     default/task-pv-claim   manual                   2m
```
```
$ kubectl get pvc task-pv-claim
NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
task-pv-claim   Bound     task-pv-volume   10Gi       RWO           manual         30s
```
### 8.4. Create a Pod
```
kind: Pod
apiVersion: v1
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
       claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx:1.7.9
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
### 8.5. Check what nginx is serving
Exec into the pod and curl localhost
```
$ kubectl exec -it task-pv-pod -- /bin/bash
  $ apt-get update
  $ apt-get install curl
  $ curl localhost
Hello from Kubernetes storage
```

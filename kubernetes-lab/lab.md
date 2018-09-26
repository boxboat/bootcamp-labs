1.  Remove taint
`kubectl taint nodes --all node-role.kubernetes.io/master-`

2.  Create Deployment

vim deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-world-deploy
  labels:
    app: hello-world
spec:
  replicas: 6
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: nginx
        ports:
        - containerPort: 80
```
kubectl apply -f deployment.yaml

3.  Expose deployment
```kubectl expose deployment/hello-world-deploy --type="NodePort"```
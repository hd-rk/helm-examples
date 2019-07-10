### v3-condition example

`parent` chart has dependency on `child` with condition
```
dependencies:
- name: child
  version: 0.1.0
  repository: file://../child
  condition: child.enabled
```

Helm 3 (`version.BuildInfo{Version:"v3.0.0-alpha.1", GitCommit:"b9a54967f838723fe241172a6b94d18caf8bcdca", GitTreeState:"clean"}`) seems to ignore the condition when running `helm install`

```
helm install test charts/parent --set child.enabled=false --dry-run
```
`child` artifacts are still rendered
- `parent/charts/child/templates/service.yaml`
- `parent/charts/child/templates/deployment.yaml`

Output:

```
NAME: test
LAST DEPLOYED: 2019-07-10 16:03:05.230221 +0800 +08 m=+1.455052302
NAMESPACE: test
STATUS: pending-install

MANIFEST:
---
# Source: parent/charts/child/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-child
  labels:
    app.kubernetes.io/name: child
    helm.sh/chart: child-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: child
    app.kubernetes.io/instance: test
---
# Source: parent/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-parent
  labels:
    app.kubernetes.io/name: parent
    helm.sh/chart: parent-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: parent
    app.kubernetes.io/instance: test
---
# Source: parent/charts/child/templates/deployment.yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: test-child
  labels:
    app.kubernetes.io/name: child
    helm.sh/chart: child-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: child
      app.kubernetes.io/instance: test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: child
        app.kubernetes.io/instance: test
    spec:
      containers:
        - name: child
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: parent/templates/deployment.yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: test-parent
  labels:
    app.kubernetes.io/name: parent
    helm.sh/chart: parent-0.1.0
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: parent
      app.kubernetes.io/instance: test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: parent
        app.kubernetes.io/instance: test
    spec:
      containers:
        - name: parent
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}

NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods -l "app=parent,release=test" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

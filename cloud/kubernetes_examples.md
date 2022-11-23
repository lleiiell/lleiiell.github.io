## Kubernetes 示例

### Pod

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

### Deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-deployment
  template:
    metadata:
      labels:
        app: hello-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### ReplicaSet

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-replicaset
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### Service

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod4service
  labels:
    app: hello-pod4service
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello-pod4service
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 60']
      restartPolicy: OnFailure
    # The pod template ends here
```

输出

```bash
k get jobs
# NAME        COMPLETIONS   DURATION   AGE
# hello-job   0/1           36s        36s

k get pods
# NAME              READY   STATUS    RESTARTS   AGE
# hello-job-bqnrp   1/1     Running   0          18s


k get jobs
# NAME        COMPLETIONS   DURATION   AGE
# hello-job   1/1           77s        94s

k get pods
# NAME              READY   STATUS      RESTARTS   AGE
# hello-job-bqnrp   0/1     Completed   0          87s

```

### StatefulSet

```yml
apiVersion: v1
kind: Service
metadata:
  name: hello-statefulset
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: hello-statefulset
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

### ConfigMap

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-configmap
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    
```

### CRD

crontab-crd.yaml

```yml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```

my-crontab.yaml

```yml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: hello-crontab
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

### Secret

mysecret

```yml
apiVersion: v1
kind: Secret
metadata:
  name: hello-secret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

envFrom

```yml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod4secret
spec:
  containers:
    - name: test-container
      image: busybox:1.28
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: hello-secret
  restartPolicy: Never
```

输出

```bash
k logs hello-pod4secret 
# PASSWORD=1f2d1e2e67df
# USER_NAME=admin
```

### 参考

- https://kubernetes.io/docs/concepts/workloads/pods/
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- https://kubernetes.io/docs/concepts/services-networking/service/
- https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
- https://kubernetes.io/docs/concepts/configuration/secret/

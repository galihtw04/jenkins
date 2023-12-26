# Install jenkins on kubernetes

verify node
```
kubectl get node -o wide
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/3cf44aea-b921-4698-9c48-84520a2e6b2d)

```
mkdir ~/jenkins && cd ~/jenkins
```

- create namespace
```
kubectl create namespace devops-tools
```

```
kubectl get ns
```


- create ServiceAccount
```
cat << 'EOF' > serviceaccount.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: devops-tools

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: devops-tools
EOF
```

```
kubectl apply -f serviceaccount.yaml
```

```
kubectl get clusterrole,clusterrolebindings,serviceaccounts -A | grep jenkins
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/f675a9ca-e438-4531-b946-f30e045b78ea)


- create volume
```
cat << 'EOF' > pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: devops-tools
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
EOF
```
> karena pada cluster saya sudah memiliki nfs provisioner kita hanya perlu buat PVC

```
kubectl apply -f
```

```
kubectl get pvc -n devops-tools
ls /nfs-share/
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/2005ba32-1896-4e41-b1ea-8832ffcb3bee)

- create deployment
```
cat << 'EOF' > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-server
  template:
    metadata:
      labels:
        app: jenkins-server
    spec:
      securityContext:
            fsGroup: 1000
            runAsUser: 1000
      serviceAccountName: jenkins-admin
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          resources:
            limits:
              memory: "2Gi"
              cpu: "1000m"
            requests:
              memory: "500Mi"
              cpu: "500m"
          ports:
            - name: httpport
              containerPort: 8080
            - name: jnlpport
              containerPort: 50000
          livenessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 90
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          volumeMounts:
            - name: jenkins-data
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-data
          persistentVolumeClaim:
              claimName: jenkins-pv-claim
EOF
```

```
kubectl apply -f deployment.yaml
```

```
kubectl get deployments -n devops-tools
kubectl get pods -n devops-tools
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/88c9251e-273a-4e9f-a9c5-5ecf9686eda6)


- create service nodeport
```
cat << 'EOF' > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: devops-tools
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/path:   /
      prometheus.io/port:   '8080'
spec:
  selector:
    app: jenkins-server
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
EOF
```

```
kubectl apply -f service.yaml
```

```
kubectl get svc -n devops-tools
```
![image](https://github.com/galihtw04/jenkins/assets/96242740/74d3d5df-bff8-4b4c-85c7-fb1030d660ef)

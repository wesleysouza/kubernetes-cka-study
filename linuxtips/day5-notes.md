# Day 5

## Ingress

É uma maneira de abstrair os seus services e expor cada um deles na Internet. O Ingress cria rotas e faz roteamento para os usuários acessarem os serviços.

Ingress controler do nginx.

**Default backend**: local retornando quando o usuário pede uma aplicação inexistente.

Crie o arquivo app1.yaml com o conteúdo abaixo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: dockersamples/static-site
        env:
        - name: AUTHOR
          value: GIROPOPS
        ports:
        - containerPort: 80
```

Agora crie o arquivo app2.yaml com as mesmas definições do app1. Após criar os arquivos com os seus respectivos conteúdos crie-os com o comando abaixo:

```
kubectl create -f app1.yaml
kubectl create -f app2.yaml
```

**Agora vamos criar os services**

Crie o arquivo svc-app1.yaml com o conteúdo abaixo:

```yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: appsvc1
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: app1 
```

Obs.: O campo **selector** serve para "amarrar" esse service ao deployment definido anteriormente.

Crie uma cópia do arquivo svc-app1.yaml com o nome svc-app2.yaml. Após isso altere os campos de svc-app2.yaml para que ele seja utilizado na aplicação 2. 

Agora crei os services com o comando abaixo:

```
kubectl create -f svc-app1.yaml
kubectl create -f svc-app2.yaml
```

Verifique se está tudo ok com o comando abaixo:

```
kubectl get deploy,svc,po
```

Vendo os endpoints

```
kubectl get endpoints
```

### Criando o default-backend

Crie o arquivo **default-backend.yaml** com a definição abaixo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: default-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: default-backend
  template:
    metadata:
      labels:
        app: default-backend
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: default-backend
        image: gcr.io/google_containers/defaultbackend:1.0
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 10m
            memory: 20Mi
          requests:
            cpu: 10m
            memory: 20Mi
```

Crie o namespace ingress:

```
kubectl create namespace ingress
```

Crie o novo Deployment no namespace ingress:

```
kubectl create -f default-backend.yaml -n ingress
```

### Criando o default-backend-service 

Crie o arquivo **default-backend-service.yaml** com a definição abaixo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-backend
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: default-backend
```

Crie o service no namespace ingress:

```
kubectl create -f default-backend-service.yaml
```

Agora veja os endpoints no namespace ingress:

```
kubectl get ep -n ingress
```

### Criando um ConfigMap

Agora crie o um arquivo para definir um configMap a ser utilizado pela nossa aplicação:

```
vim nginx-ingress-controller-config-map.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-ingress-controller-conf
  labels:
    app: nginx-ingress-lb
data:
  enable-vts-status: 'true'
```

Crie o configMap:

```
kubectl create -f nginx-ingress-controller-config-map.yaml -n ingress
```

Criando o configMap:

```
kubectl create -f nginx-ingress-controller-config-map.yaml -n ingress
```

Visualize o configMap no namespace ingress:

```
kubectl get configmaps -n ingress
```

### Criando o Service Account

Crie o arquivo **nginx-ingress-controller-service-account.yaml** com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx
  namespace: ingress
```

## Criando o ClusterRole

Crie o arquivo **nginx-ingress-controller-cluster-role.yaml** com o conteúdo abaixo:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nginx-role
rules:
- apiGroups:
  - ""
  - "extensions"
  resources:
  - configmaps
  - secrets
  - endpoints
  - ingresses
  - nodes
  - pods
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - list
  - watch
  - get
  - update
- apiGroups:
  - "extensions"
  resources:
  - ingresses
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
- apiGroups:
  - "extensions"
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
  - create
```

### Criando ClusterRoleBiding

Associar o Cluster Role com o usuário.

Crie o arquivo **nginx-ingress-controller-cluster-rolebiding.yaml** com o conteúdo abaixo:

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nginx-role
  namespace: ingress
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-role
subjects:
- kind: ServiceAccount
  name: nginx
  namespace: ingress
```

Crie os objetos:

```
kubectl create -f nginx-ingress-controller-service-account.yaml -n ingress
kubectl create -f nginx-ingress-controller-cluster-role.yaml -n ingress
kubectl create -f nginx-ingress-controller-clusterrolebinding.yaml -n ingress
```

### Criando o Deployment do nginx-ingress-controller

Crie o arquivo **nginx-ingress-controller-deployment.yaml** com o conteúdo abaixo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress-lb
  revisionHistoryLimit: 3
  template:
    metadata:
      labels:
        app: nginx-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      serviceAccount: nginx
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.9.0
          imagePullPolicy: Always
          readinessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
          livenessProbe:
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            timeoutSeconds: 5
          args:
            - /nginx-ingress-controller
            - --default-backend-service=ingress/default-backend
            - --configmap=ingress/nginx-ingress-controller-conf
            - --v=2
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - containerPort: 80
            - containerPort: 18080
```

Crie o objeto:

```
kubectl create -f deployments -n ingress
```

Ver todos os pods:

```
kubectl get pods --all-namespaces
```

### Criando um objeto do tipo Ingress

```
vim nginx-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: 192.168.0.100 # Mude para o seu endereço dns
    http:
      paths:
      - backend:
          service:
            name: nginx-ingress
            port:
              number: 18080
        path: /nginx_status
        pathType: Prefix
```

Agora crie um arquivo para definir o ingress que redirecionará para os serviços das aplicações que criamos no início desta seção:

```
vim app-ingress.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: app-ingress
spec:
  rules:
  - host: ec2-54-198-119-88.compute-1.amazonaws.com # Mude para o seu endereço dns
    http:
      paths:
      - backend:
          service:
            name: appsvc1
            port:
              number: 80
        path: /app1
        pathType: Prefix
      - backend:
          service:
            name: appsvc2
            port:
              number: 80
        path: /app2
        pathType: Prefix
```

### Criando o service

Crie um arquivo para definir um service do tipo nodePort:

```
vim nginx-ingress-controller-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30000
      name: http
    - port: 18080
      nodePort: 32000
      name: http-mgmt
  selector:
    app: nginx-ingress-lb
```






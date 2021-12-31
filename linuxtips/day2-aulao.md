# Day 2 - Aulão

## Intro

Ativando o completion bash:

```
source <(kubectl completion bash)
```

Criando um pod a partie de um arquivo de definição YAML:

```
kubectl get pods nginx -o yaml > new-pod.yaml
```

Criando com o dry-run:

```
kubectl run new-nginx --image nginx --dry-run=client -o yaml > new-pod-template.yaml
```

O dry-run evita muitas definições desnecessárias e é muito importante para o exame!

Crie o pod:

```
kubectl create -f nome-do-pod.yaml
```

## Services

Criando um serviço:

```
kubectl expose pod nome-pod --port 80
```
Verifique os serviços rodando

```
kubectl get services 
```

Ver mais detalhes do service:

```
kubectl describe service nome-so-service
```

Delete o service!

### Cliando Service ClusterIP

Criando o Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  selector:
    matchLabels:
      app: my-deployment
  replicas: 2
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Criando o service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-deployment
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Crie o serviço:

```
kubectl create -f meu-primeiro-svc-clusterip.yaml
```

É possível definir dentro do **spec** um sessionAffinity do modo demostrado abaixo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-deployment
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  sessionAffinity: ClientIP
```

O **sessionAffinity** dar afinidade para o cliente utilizar o mesmo pod sempre que fizer uma requisição.

Observe as informações do serviço:

```
kubectl describe service nome-do-service
```

### Criando um service NodePort

Criando pela linha de comando:

```
kubectl expose pod nome-do-pod --type=NodePort --port 80
```

Criando um pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-pod
  name: my-pod
spec:
  containers:
  - name: my-pod
    image: nginx
    ports:
    - containerPort: 80
```

Criando o service NodePort para esse pod:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: my-pod
  name: my-pod
spec:
  ports:
  - nodePort: 31111
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: my-pod
  type: NodePort
```

#### Endpoints

Deu problema? Verifique se foi criado o endpoint com o comando:

```
kubectl get endpoints
```

Também é possível ver uma descrição mais detalhada do endpoint:

```
kubectl describe endpoints nome-do-servico
```

### Criando um LoadBalancer

Criando o LoadBalancer pela linha de comando:

```
kubectl expose pod nginx --type LoadBalancer --port 80
```

É necessária a integração com a cloud ou outro serviço de load balçancer para que ele funcione.

Criando o arquivo de manifesto do serviço LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: my-pod
  name: my-pod
spec:
  ports:
  - nodePort: 31111
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: my-pod
  type: LoadBalancer
```

Crie o serviço e verifique se ta ok.



### Trabalhando com Deployments

Criando deployment a partir da linha de comando:

```
kubectl create deployment --image=nome-imagem nome-deployment
```

Ver os deployments:

```
kubectl get deployments.apps
```

Auemntando o número de pods:

```
kubectl scale deployment --replicas=n nome-deployment
```

Expondo o deployment:

```
kubectl expose deployment nome-deployment --port=80
```

## Limitando Recursos

Sempre devemos limitar os recursos de nossas aplicações.

Limits e Requests:
- Limits: Máximo que poderá ser utilizado;
- Requests: Mínimo garantido.

O recurso de cpu é medido em milicore. Desse modo, 200m é igual a 20% da CPU.

## Namespace

É como se fosse criar clusters virtuais.

Uma coisa interessante é limitar os recursos por namespace com o LimitRange.

## Taint

É um forma de adicionar propriedades nos nós do seu cluster.

Tipos:
- NoSchedule: Nada novo vai ser adicionado nesse no.
- NoExecute: Nada em execução no nó.

Adicionando Taint em todos os nós:

```
kubectl taint node --all key1=value1:NoSchedule
```

Removendo:

```
kubectl taint node --all key1=value1:NoSchedule-
```

### Cordon e Uncordon

```
kubectl cordon nome-do-no
```

```
kubectl uncordon nome-do-no
```


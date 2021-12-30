# Day 2

## Introdução

É uma forma de pegar o deployment e permitir que ele seja acessado de fora do cluster. 

Afirmações importantes sobre a arquitetura:
- Controler de um POD é o ReplicSet;
- Controller do ReplicaSet é o Deployment;
- Se um POD morrer o ReplicaSet é o controller responsável por subir outro.
- Atualização de um ReplicaSet (atualização de uma imagem, alteração do deployment), derruba o ReplicaSet atual e sobe um novo. O antigo permacene para situações de roolback.
- Namespace: separação do cluster por grades. No namespace é possível aplicar cotas.
- KubeAPIServer: responsável por toda a comunicação.
- KubeProxy: responsável por gerenciar a rede dos containers, liberação de portas e etc.

CNI: O Kubernetes não vai fazer a sua rede funcionar por completo... Toda a comunicação de rede entre os Pods é por IP, não usa NAT. O CNI é uma biblioteca para que vc faça o eu plugin para que a rede funcione (rede do container). O wevenet é um desses plugins.

O WaveNet atua na camada 2 e usa o VXLAN.

## Services

Camada que redireciona todas as requisições de fora para o nosso deployment.

### Criando o deployment com o dry-run

```
kubectl create deployment primeiro_deployment --image=nginx --port=80 --dry-run=client
 -o yaml > primeiro_deployment.yaml
```

Editando o arquivo e alterando o número de réplicas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: primeiro-deployment
  name: primeiro-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: primeiro-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: primeiro-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
```
Varificando o deployment:

```
kubectl get deployments.apps
```

### Subindo "na mão"

```
kubectl run nginx --image nginx --port=80
```

Esse comando run vai ser depreciado, ou já ta.

### Criando service para um deployment

```
kubectl expose deployment nome-deployment
```

Verifique o service criado: 

```
kubectl get service
```

O tipo do service criado é ClusterIP, só acessado de dentro do cluster. Primeiramente sempre vai ser criado o cluster IP.

Verificando os endpoints:

```
kubectl get endpoints
```

O endpoint é o ip do meu pod.

Escalando réplicas para testar:

```
kubectl scale --replicas=10 deployment nome-deployment
```

Testanto com o curl:

```
curl + ip do endpoint
```

Vendo os logs:

```
kubectl logs -f id_pod
```

### Criando service a partir de um arquivo

Verifique os services que estão rodando:

```
kubectl get services
```

Redirecionando a conf yaml de um server para um arquivo:

```
kubectl get services nome-do-service -o yaml > meu-primeiro-service.yaml
```
Conteúdo do arquivo após modificado:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: primeiro-deployment
  name: primeiro-deployment
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: primeiro-deployment
  sessionAffinity: None
  type: ClusterIP
```

#### Deletando e recriando o serviço

Vamos deletar e recriar o serviços a partir do arquivo:

```
kubectl delete service nome-do-service
```

Recriando o service a partir do arquivo:

```
kubectl create -f meu-primeiro-service.yaml
```

Verificando o service:

```
kubectl get services
```

Caso o service esteja up, ta tudo certo! 

### Trabalhando com o NodePort

```
kubectl expose deployment nome-do-deployment --type=NodePort
```

Verifique os services:

```
kubectl get services
```

Verificando as informações do service

```
kubectl describe service nome-do-service
```

Redirecionando a conf NodePort do serviço para um arquivo yaml:

```
kubectl get services primeiro-deployment -o yaml > nome-do-service-node-port.yaml 
```

Arquivo após modificações:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: primeiro-deployment
  name: primeiro-deployment
  namespace: default
spec:
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 32222
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: primeiro-deployment
  sessionAffinity: None
  type: NodePort
```

Agora delete o service anterior e recrie a partir deste arquivo.

## Criando um Load Balancer

Para criar um deployment tipo Load Balancer utilize o comando:

```
kubectl expose deployment nome-deployment --type=LoadBalancer
```

Verificando o serviço criado:

```
kubectl get services
```

### Criando o arquivo de configuração YAML

```
kubectl get services nome-do-service -o yaml > meu-primeiro-service-load-balancer.yaml
```

Arquivo após modificações:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: primeiro-deployment
  name: primeiro-deployment
  namespace: default
spec:
  allocateLoadBalancerNodePorts: true
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 33333
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: primeiro-deployment
  sessionAffinity: None
  type: LoadBalancer
```

Crie o novo serviço com o comando abaixo:

```
kubectl create -f meu-primeiro-service-load-balancer.yaml
```

Agora verifique se está tudo ok no services:

```
kubectl get services
```

Sempre que um service é criado cria-se também um endpoint.

Obs.1: Use o des
cribe para analisar os objetos deployados.

Obs.2: A saída dos comandos é muito importante na prova de certificação.
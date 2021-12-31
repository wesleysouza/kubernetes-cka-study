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

### Resumo

Tipos de Service:
- ClusterIP;
- NodePort:
  - Quando um NodePort é criado junto com ele vem um ClusterIP.
- LoadBalancer:
  - Quando um LoadBalancer é criado junto com ele vem um ClusterIP e NodePort.

## Limitando Recursos

Sempre que subir uma aplicação vc deve limitar a quantidade de recursos que essa aplicação pode consumir.

Para limitar vamos editar um arquivo de deployment no campo **resources**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deploy-limitado
  name: deploy-limitado
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deploy-limitado
  strategy: {}
  template:
    metadata:
      labels:
        app: deploy-limitado
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 512Mi
            cpu: "500m"
          requests:
            memory: 256Mi
            cpu: "250m"
status: {}
```

Diferenças entre limits e requests:
- Limits: Total reservado;
- Requsts: Mínimo de recursos garantidos;]

### Criando o "Deploy Limitado"

```
kubectl create -f deploy-limitado.yaml
```

Vendo os deployments em execução:

```
kubectl get deployments
```

Analisando com o describe:

```
kubectl describe deployments. deploy-limitado
```

Obs.: É possível substituir o deployment antigo com o comando **replace**:

```
kubectl replace -f nome-deployment
```
#### Criando o service parao "Deploy limitado"

É possíver editar um serviço "no quente" com o comando abaixo:

```
kubectl edit service nome-so-serviço
```

Vai abrir um arquivo pelo vim para editar e quando for salvo o serviço será editado. É possível mudar o número de replicas, tipo do serviço e etc.

Para ver o número de replicas use o comando:

```
kubectl get replicasets.
```

Obs.: Modificações incorretas ou modificações não permitidas são ignoradas!

### Brincando com os pods

Vendo os pods e o local onde eles estão:

```
kubectl get pods -o wide
```

Executando coisas dentro dos containers através do bash:

```
kubectl exec -ti nome-container -- bash
```

Instalando o stress:

```
apt-get update && apt-get install -y stress
```

Testante o stress:

```
stress --vm 1 --vm-bytes 256M
```

Valores que passam dos limites como 550 vão estourar os limites e matar o processo.

### Trabalhando com namespaces

Verificando os namespaces:

```
kubectl get namespaces
```

Criando um namespace:
```
kubectl create namespace nome-do-novo-namespace
```

Verificando namespace:
```
kubectl describe namespaces nome-do-namespace
```

Pegando o arquivo yaml do namespace:

```
kubectl get namespaces aula02 -o yaml > meu_primeiro_namespace.yaml
```

Arquivo modificado:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: strigus
spec:
  finalizers:
  - kubernetes
```

Criando o namespace:

```
kubectl create -f nome-do-arquivo.yaml
```

Verifique os namespaces:

```
kubectl get namespaces
```

### Criando um LimitRange

Objeto utilizado para limitar a quantidade de determinado recursos que um namespace pode utilizar. Esse limite também pode ser aplicado a todo o cluster.

Criando um arquivo de definição do LimitRange:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limitando-recursos
spec:
  limits:
  - default:
      cpu: 1
      memory: 100Mi
    defaultRequest:
      cpu: 0.5
      memory: 80Mi
    type: Container
```

Criando o LimitRange em um namespace:

```
kubectl create -f arquivo-definição.yaml -n nome-do-namespace
```

Verificando se o LimitRange foi criado:

```
kubectl get limitranges -n nome-do-namespace
```

Vendo detalhes do LimitRange

```
kubectl describe limitranges -n nome-do-namespace nome-do-limit-range
```

#### Criando um pod "limitado" por meio de um arquivo YAML

Crie um arquivo e adicione o conteúdo abaixo:

```

```

Crie o pod:

```
kubectl create -f pod-limitado.yaml -n aula02
```

Verifique os pods que estão rodando dentro do namespace:

```
kubectl get pods -n nome-namespace
```

Vendo com mais detalhes:

```
kubectl describe pods -n nome-namespace
```

## Taints

Vemos verificar um dos nós:

```
kubectl describe nodes nome-do-no
```

Criando um novo deployment:

```
kubectl create -f nome-do-deployment.yaml
```

Verifique os pods:

```
kubectl get pods -o wide
```

Impedindo um nó de subir novos pods:

```
[SINTAXE]
kubectl taint node nome-do-no nome-da-chave=valor
```

Exemplo:

```
kubectl taint node elliot-02 key1=value1:NoSchedule
```

Verifique se funcionou subindo novos pods e verificando se o Taint está configurado com o describe.

Removendo o Taint:

```
kubectl taint node elliot-02 key1=NoSchedule-
```

Criando um Taint para não executar:

```
kubectl taint node elliot-02 key1=value1:NoExecute
```

Obs.: O NoExecute pode ser muito útil em uma manutenção.

Removendo o NoExecute:

```
kubectl taint node elliot-02 key1=NoExecute-
```

Obs.: Após remover o Taint NoExecute os pods não são balanceados entre os nós workers, para fazer isso será necessário usar o **kubectl scale**, reduzindo e depois aumentando o número de réplicas, exemplo:

```
kubectl scale --replicas=1 deployment nome-do-deployment
```

Agora vamos aumentar:

```
kubectl scale --replicas=6 deployment nome-do-deployment
```

Verificando o balanceamento:

```
kubectl get pods -o wide
```


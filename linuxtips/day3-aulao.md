# Day 3 Aulão

## Deployments

O Deployment é um resource com responsabilidade de instruir o Kubernets a criar e atualizar a saúde das instâncias de suas aplicações.

Vamos iniciar ativando o completion bash

```
source <(kubectl completion bash)
```

Criando o arquivo manifesto de um Deployment utilizando o **dry-run**:

```
kubectl create deployment --dry-run=client -o yaml --image=nginx nome-do-deployment > meu-deployment.yaml
```

Copie e cole o primeiro e segundo deployments do livro e depois os crie.

Utilize o describe para analisar os deployments:

```
kubectl describe deployments.apps primeiro-deployment
```

### Filtrando por labels:

Obtendo todos os pods com a label dc=UK:

```
kubectl get pods -l dc=UK
```

Adicionando uma coluna com a opção **-L**:

```
kubectl get pods -L dc
```

### Adicionando labels

É possível adicionar labels em diferentes recursos do seu cluster. O comando abaixo demostra os tipos de labels que podem ser adicionados:

```
kubectl label + 2 tabs
```

Adicionando label em um nó:

```
kubectl label nodes nome-do-no label
```

Exemplo:

```
kubectl label nodes k8s-worker1 dc=UK
```

Removendo o label:

```
kubectl label nodes k8s-worker1 dc-
```

## NodeSelector

O NodeSelector é uma forma de classificar os nós. Desse modo, é possível definir onde as aplicações vão rodar de acordo com suas características e a do cluster.

Adicionando label a um nó:

```
kubectl label node nome-do-no disk=SSD
```

Para sobrescrever a labl disk use o comando abaixo com a opção **--overwrite**:

```
kubectl label nodes nome-do-no disk=HDD --overwrite
```

Listando todas as labels de um nó:

```
kubectl label nodes nomde-do-no --list
```

Listando todas as labels de um pod de um deployment:

```
kubectl label pod nome-deployment --list
```

Listando todas as labels de um deployment:

```
kubectl label deployments.apps nome-deployment --list
```

Crie o arquivo terceiro-deployment.yaml com o conteúdo abaixo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: terceiro-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx
        dc: Netherlands
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx2
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      nodeSelector:
        disk: SSD
```

Verifique onde os pods desse deployment estão rodadando:

```
kubectl pods nome-deployment -o wide
```

Após modificar um arquivo de recurso podemos aplicar essas moficicações com o **apply**:

```
kubectl apply -f nome-deployment.yaml
```

O NodeSelector é importante pois possibilita que as aplicações aproveitem as características do cluster (varacterística de cada nó). Além disso, é muito importante aplicar os limites de rosources (CPU, RAM, IO).

## Kubectl edit

O **kubectl edit** possibilita a modificação de qualquer objeto do kubernetes. Vamos editar um deployment:

```
kubectl edit deployments.apps terceiro-deployment
```

Com esse comando será aberto o vim com a definição do arquivo. Sendo assim, é só modificar e salvar que as modificações vão ser aplicadas.

Obs.: Essas modificações não vão ficar salvas no arquivo de manifesto.

## ReplicaSet

Garante a quantidade de pods e reursos necessários para um Deployment (trabalha com as réplicas).

**Obs.: Não é recomendado que você mesmo crie os seus replicaSets, o ideal é criar o Deployment e ele que fica responsável por controllar esse objeto**.

**Obs.: A criação desse ReplicaSet é apenas para fins didáticos**. 

Criado o nosso primeiro arquivo manifesto de um ReplicaSet:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-set-primeiro
spec:
  replicas: 3
  selector:
    matchLabels:
      system: Giropops
  template:
    metadata:
      labels:
        system: Giropops
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Criando o ReplicaSet:

```
kubectl create -f primeiro-replicaset.yaml
```

Vendo os ReplicaSets:

```
kubectl get replicasets.apps
```

## DeamonSet

Semelhante ao ReplicaSet, nesse caso você não especifica a quatidade de réplicas. Ele é bem interessante para serviços que necessitem rodar em todos os nodes do cluster, como por exemplo, coletores de logs e agentes de monitoração.

Conteúdo do arquivo do nosso primeiro DeamonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemon-set-primeiro
spec:
  selector:
    matchLabels:
      system: Strigus
  template:
    metadata:
      labels:
        system: Strigus
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Criando o DeamonSet:

```
kubectl create -f deamon-set.yaml
```

O nó master não receberá pods do deamon-set devido ao Taint!

Pegando os pods com a label **system=Stringus**:

```
kubectl get pods -o wide -l system=Strigus
```

## Rollouts e Rollbacks

Vendo o histórico de coisas que foram alteradas com o rollout:

```
kubectl rollout history deployment primeiro-deployment
```

Esse comando vai exibir uma saída semelhante a que está apresentada abaixo:

```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Visualizando a revisão 1 com mais detalhes:

```
kubectl rollout history deployment nome-deployment revision=1
```

Fazendo o rollback:

```
kubectl rollout undo deployment --to-revision 1
```

Obs.: É possível especificar o número máximo de revisões que vão ser armazenadas.

Vendo o status do rollout:

```
kubectl rollout status deployment nome-deployment
```

Adicionando estratégia de update:

```yaml
spec:
  ...
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```

Parâmetros:
- **maxUnavailable**: Número máximo de pods que podem ficar indisponíveis durante a atualização.


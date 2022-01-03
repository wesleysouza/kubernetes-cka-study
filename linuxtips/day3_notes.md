# Day 3

## Intro

Tópicos da aula:
- Deployment;
- Labels
- NodeSelector
- Kubectl edit
- ReplicSet
- DeamonSet
- Rollouts e Rollbacks
- Volumes

Qual problema a sua empresa está resolvendo com o Kubernetes?

## Deployment

O Deployment é o controller do ReplicaSet, quando um Deployment é criando junto com ele vem um ReplicaSet.

Crie um arquivo manifesto de Deployment com o conteúdo abaixo:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: nginx
    app: giropops
  name: primeiro-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
        dc: UK
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
```

Agora crie o Deployment com o comando:

```
kubectl create -f nome-do-arquivo.yaml
```

Verifique o Deployment:

```
kubectl get deployments
```

Crie um segundo arquivo de Deployment alterando o campo DC.

Verifique as flags com o describe:

```
kubectl describe deployments nome-deployment
```

As flags run e app.

## Deployment, Label e Node Selector

Pegando todos os pods que estão rodando em NL:

```
kubectl get pods -l dc=NL
```

Adicionando uma coluna na saída do get:

```
kubectl get pods -L dc
```

Os labels pode ser colocados onde vc quiser!

Adicionando o label **disk=SSD** em um node:

```
kubectl label nodes nome-do-no disk=SSD
```

Ver todos os labels atribuídos a um nó:

```
kubectl label nodes nome-do-no --list
```

Sobrescrevendo um label:

```
kubectl label nodes nome-do-no disk=HDD --overwrite
```

### NodeSelector

É possível, através dos labels e o NodeSelector selecionar um nó específico para rodar o nosso deployment.

Crie um terceiro deployment e adicione o conteúdo abaixo no arquivo:

```yaml
spec:
...
  spec:
  ...
    nodeSelector:
      disk: SSD
```

Nesse caso, o nosso deployment só vai subir se existir um nó com essa flag.

#### Verificando se o deployment subiu no nó específico

Verifique as flags dos nós e se algum deles atende os requisitos definidos no NodeSelector do Deployment.

Veja onde os pods do deployment subiram com o comando baixo:

```
kubectl get pods -o wide
```

Qualquer alteração no nodeSelector cria um novo deployment.

#### Removendo labels

Recomendo todas as labels **dc** de todos os nós:

```
kubectl label dc- --all
```

Removendo as labels **disk** de todos os nós:

```
kubectl label disk- --all
```

### ReplicaSet

O Deployment é o controler do ReplicaSet.

Criando o arquivo de definição do ReplicaSet:

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: replica-set-primeiro
spec:
  replicas: 3
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

Vendo o ReplicaSet:

```
kubectl get replicaset 
```

Outros comandos:

```
kubectl describe rs replica-set-primeiro

kubectl delete pod replica-set-primeiro-6drmt

kubectl get pods -l system=Giropops

kubectl edit rs replica-set-primeiro

kubectl get pods -l system=Giropops

kubectl get deployment

kubectl delete pod replica-set-primeiro-7j59w

kubectl describe pod replica-set-primeiro-xzfvg

kubectl get rs

kubectl delete rs replica-set-primeiro
```

## DeamonSet

ReplicatSet com a diferença que você não escolhe quantas réplicas ele vai ter. O DeamonSet vai ser utilizado quando um pod precisa rodar em todos os nós do cluster. Por exemplo, um prometheus exporter coleta dados de todos os nós do cluster, então quando esse cara subir é preciso garantir que ele está coletando informações de todos os nós.

Cirando o arquivo de definição do DeamonSet:

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
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
  updateStrategy:
    type: RollingUpdate
```

Criando o DeaonSet:

```
kubectl create -f primeiro-deamonset.yaml
```

Verificando se o DeamonSet subiu:

```
kubectl get ds
```

Veja onde os pods estão com o comando:

```
kubctl get pods -o wide
```

O DeamonSet não sobe no Master por causa do Taint.

Removendo o taint do master:

```
kubectl taint node nome-do-master node-role.kubernetes.io/master-
```

Agora podemos subir outros pods no master, porém isso não é bom. Retornando para o estado anterior:

```
kubectl taint node nome-do-no nod-role.kubernetes.io/master=value1:NoSchedule
```

Outros comandos:

```
kubectl taint nodes --all node-role.kubernetes.io/master-

kubectl create -f  primeiro-daemonset.yaml

kubectl get daemonset

kubectl describe ds daemon-set-primeiro

kubectl get pods -o wide

kubectl set image ds daemon-set-primeiro nginx=nginx:1.15.0

kubectl describe ds daemon-set-primeiro

kubectl delete pod daemon-set-primeiro-jh2sp
```

## Rollouts e Rollbacks

### Rollout

Exibindo o histórico de modificações do deamon-set-primeiro:

```
kubectl rollout history deamonset deamon-set-primeiro
```

O comando anterior vai mostrar todas as revisões (versões) do objeto da seguinte forma:

```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

É possíver ver mais detalhes de cada uma das revisões com o comando abaixo:

```
kubectl rollout deamonset deamon-set-primeiro --revision=1
```

Fazendo o rollout para a versão 1:

```
kubectl rollout undo deamoset deamon-set-primeiro --to-revision=1
```

Obs.: Na definição do objeto é necessário passar uma estratégia de update para que o **rollout** funcione!

Exemplo de update strategy em um yaml file:

```yaml
spec:
  ...
  updateStrategy:
    type: RollingUpdate
```

Agora podemos editar o objeto e fazer o **rollout**.

Verifique se está tudo ok observando a saída do objeto no formato yaml:

```
kubectl get deamonsets deamon-set-primeiro -o yaml
```

Exercício: Crie um DeamonSet modifique com o edit e depois faça o rollout.

## Canary Deploy

É possível rodar duas versões da mesma aplicação em um único service. Esse comportamente pode ser aproveitado para testar novas versões de uma aplicação.

Faça clone do repositório no github:

```
 git clone https://github.com/badtuxx/k8s-canary-deploy-example.git
```

Copie os arquivos para deploy:

```
cp k8s-canary-deploy-example/deploy-app-v1-playbook/roles/common/files/* .
cp k8s-canary-deploy-example/canary-deploy-app-v2-playbook/roles/common/files/* .
```

## Mais sobre Rollouts e Rollbacks

```yaml
spec:
  ...
  strategy:
    type: RollingUpdate
    maxSurge: 2
    maxUnavailable: 3
```

Definições:
- **maxSurge**: Durante o processo de update será permitido a adição de até mais 2 containers durante o update.
- **maxUnavailable**: Número máximo de containers fora durante o update.

Vero o status do deployment:

```
kubectl rollout status deployment nome-deployment
```




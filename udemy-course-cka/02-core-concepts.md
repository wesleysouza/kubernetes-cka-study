# 2 Core Concepts

## 2.20 PODs with YAML

Estrutura geral de um manifesto yaml:

```yaml
apiVersion:
kind:
metadata:

spec:
```

Essas são as propriedades raiz do manifesto. Esses campos são obrigatórios nos arquivos de configuração.

Versões do apiVersion:

| Kind       | Version |
|------------|---------|
| POD        | v1      |
| Service    | v1      |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |


- kind: tipo do objeto.
- metadata: dados sobre o objeto como as labels. Os dados do metadata são um dicionário.

Exemplo com labels:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
```

O campo **spec** é onde vamos definir a especificação do nosso obejto. Dentro de **spec** vamos definir os containers (também é um dicionário). Esses containers são definidos no campo **containers** na forma de Lista/Array no Yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

O Símbolo **-** na linha **- name: nginx-container** indica o primeiro item da lista.

## 2.27 ReplicaSets

Oc controladores são parte do "cérebro" do k8s, eles monitoram os objetos e fazem com que respondam de acordo com o que foi especificado.

### Replication Controller

O ReplicationControler garante que o número específicado de pods vão continuar rodando idependente de falhas que possam ocorrer. Além disso, permite que "escalemos" nossa aplicação em mais pods para atender a uma demanda crescente.

### ReplicaSets

O Replication Controller e o ReplicaSet são objetos diferentes com propósitos diferetes. O Replication Controller está sendo substituído pelo ReplicaSet.

Criando um ReplicaSet:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:

    matadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabel:
      type: front-end
```

#### Labels e Selectors 

Os labels possibilitam que os ReplicaSets filtrem e identifiquem os pods para relizar o monitoramento e tomada de decisão.

Para atualizar o número de réplicas podemos editar o arquivo de definição e depois disso executar o comando:

```
kubectl replace -f replicaset-definition.yaml
```

Outras formas:

```
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset my-replicaset
```

Para deletar o ReplicaSet:

```
kubectl delete replicaset myapp-replicaset
```

## 2.30 Deployments

O Deployment possibilita atulizar as instâncias subjacentes perfeitamente usando roling updates.

Um dos tipos de upgrade é conhecido como rolling updates.

Para ver todos os objetos criados:

```
kubectl get all
```

## 2.34 Namespaces

Para saber o número de namespaces digite o comando abaixo:

```
kubectl get namespaces
```

É possível atribuir cotas aos namespaces.

### DNS

db-service.dev.svc.cluster.local
(Service name).(Namespace).(Service).(Domain)

Em arquivos de definição podemos definir o namespace onde o objeto vai subir definindo na seção metadata o campo namespace: **nome-do-namespace**.

### Trocando de contexto

Para trabalhar em um namespace específico use o kubeconfig da maneira apresnetada abaixo:

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

Após digitar o comando acima, estaremos no namespace dev.

### Resource Quota

É possível por meio de arquivos de manifesto definir Quotas para os namespaces, veja abaixo um exemplo de arquivo para definição de Quota:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

Para criar essa Quota basta usar o comando abaixo:

```
kubectl create -f compute-qouta.yaml
```

## 2.36 Solution namespaces

```
kubectl get ns --no-headers | wc -l
kubectl -n research get pods --no-headers
kubectl get pods --all-namespaces | grep blue
```

## 2.37 Services

```

```


```

```


```

```

```

```

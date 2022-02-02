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

```

```
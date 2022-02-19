# Bonde do CKA - Aula 09

Na prova haverá vários "contextos" diferentes, ou seja, vários clusters diferentes. 

## Questão 1

Hoje o nosso gerente pediu para que fiquemos confortáveis com o gerencimento de contextos do nosso clusters. Ele está com medo que executemos algo em algum cluster errado, e assim deixando o nosso dia muito mais chatiante!

Verificando o contexto:

```
kubectl config get-contexts
```

No diretório **.kube/** vai ter um arquivo config que tem todas as informações do kubernetes, quem tiver esse arquivo vai ter acesso ao cluster (cuidado). No conteúdo desse arquivo tem várias informações, inclusive o contexto.

Criando um novo cluster:

```
kind create cluster --name producao --config kind-3nodes.yaml
```

O arquivo **kind-cluster.yaml** foi obtido no site do kind, é um modelo.

Conteúdo do arquivo:
```yaml
kind: Cluster 
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

Crie o novo cluster e veja os contextos:

```
kubectl config get-contexts
```

Mudando de contexto:

```
kubectl config use-context nome-do-context
```

O arquivo .kube/config contém as informações dos clusters.

### Resposta

Criamos dois clusters, para que pudessemos brincar cm os contextos. Para criar os clusters, nós utilizamos o Kind, e para criar o cluster, nós estamos utilizando um arquivo template,  conforme abaixo:

```bash
vim kind-custer.yaml
```

```yaml
kind: Cluster 
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

Comando para criar o cluster:

```
kind create cluster --config kind-cluster-1.yaml
```

Após criar os dois clusters, chegou a hora de trabalhar com os contextos.

Para visualizar os contextos, utilize o comando abaixo:

```
kubectl config get-contexts
```

Para selecionar determinado contexto, utilize:

```
kubectl config use-context CONTEXTO-DESEJADO
```

Vale lembrar que os contextos estão definidos no seu arquivo config, na maioria dos casos no **$HOME/.kube/config**.

## Questão 02

Precisamos criar um pod com o Nginx rodando no cluster lt-01, já no cluster giroposp-01, nós precisamos ter um deployment e um service deployment do nginx e um service apontando para esse deployment.
Os ods deverão ter o mesmo nome em todos os clusters.

56:38
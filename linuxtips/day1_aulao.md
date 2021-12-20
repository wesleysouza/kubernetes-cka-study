# Day 1: Aulão 

## Ferramentas e sites

### Linux

- systemd
- journalctl

## Documentação do Kubernetes

(Documentação)[https://kubernetes.io/docs/home/]

## The Twelve-Factor App

## Portas que devemos nos preocupar

- Master
    - API Server: 6443 TCP
    - etcd: 2379-2380 TCP
    - Kubelet: 10250 TCP, 10255 TCP
    - Scheduler: 10251 TCP
    - Controller Manager: 10252 TCP
- Workers
    - Kubelet: 10250 TCP, 12255 TCP
    - NodePort Services: 30000-32767 TCP
-Wave pod Network
    - 6783 e 6784 TCP

## Conceitos-chave do k8s

- **Pod**: O _pod_ é o menor objeto do k8s. Como dito anteriormente, o k8s bão trabalha com os containers diretamente, mas organiza-os dentro de pods, que são abstrações que dividem os mesmo recursos, como: endereços, cliclos de CPU e memória. Um pod, embora não seja comum, pode possuir vários containers;
- **Controller**: Um _controller_ é o objeto responsável por interagir com o API Server e orquestrar algum outro objeto. Exemplos de objetos desta classe são _Deployments_ e _Replication Controllers_;
- **ReplicaSets**: Objeto responsável por garantir a quantidade de pods em execução no nó;
- **Deployment**: É um dos principais controllers utilizados, o _Deployment_ garante que um determinado número de réplicas de um pod através de um outro controller chamado **ReplicaSet** esteja em execução nos nós _workers_ do cluster;
- **Jobs e CronJobs**: Responsáveis pelo gereniamento de tarefas isoladas ou recorrentes.

## Nmap

Pegando informações das máquinas com o nmap:
```
sudo nmap -sn <seu ip>/<mascara>
```

## Criação de instâncias na AWS

Após criar as instâncias é necessário liberar as portas no security group.

## Primeiros Commands

Obtendo informações dos nós:

```
kubectl describe node name-node
```

Obtendo o token para adição de nós:

```
kubeadm token create --print-join-command
```

Configurando o completion:

- Temporariamente:
```
source <(kubectl completion bash)
```
- De forma permanente:
```
kubectl completion bash > /etc/bash_completion.d/kubectl
```

Vendo os namespaces:
```
kubectl get namespaces
```

Pegando os **pods** de um namespace específico, nesse caso o kube-system:

```
kubectl get pods -n kube-system
```
Pegando os **pods** de um namespace específico, nesse caso o kube-system (com mais detalhes):

```
kubectl get pods -n kube-system o- wide
```

### Criando o primeiro pod

```
kubectl run --image=nginx nginx
```

Obtendo informações sobre os pods:
```
kubectl get pods
```

Vendo informações sobre o pod criado (nome nginx):
```
kubectl describe pods nginx
```

#### Criando um arquivo para a criação de um pod

É possível criar pod via linha de comando ou arquivos manifest em yaml e json.

Redirecionando a saída get pods para um arquivo:
```
kubectl get pods nginx -o yaml > meu_primeiro_pod.yaml
```

Criando pod através de um arquivo:

```
kubectl create -f meu_primeiro_pod.yaml
```

#### Criando um template com o dry-run

Criando o template:
```
kubectl run --image=nginx giros --dry-run=client -o yaml > giros_pod.yaml
```

Criando o pod através do template:

```
kubectl create -f giros_pod.yaml
```

Criando um deployment:

```
kubectl create deployment giropops --image=nginx --dry-run=client -o yaml > giros_deployment.yaml
```

Vendo o conteúdo: 

```
vim giros_deployment.yaml
```

Criando o deployment:

```
kubectl create -f giros_deployment.yaml
```

Vendo os deployments:

```
kubectl get -f deployments.apps
```

Criando um deployment dentro de um namespace:

```
kubectl create -f giros_deployment.yaml -n opa (namespace name)
```

Vendo o deployment no namespace opa:

```
kubectl get deployment -n opa
```

Obtendo ajuda (comando: kubectl explain):

```
kubectl explain deployment
```

## Criando serviços

Expondo as aplicações...

```
kubectl expose deployment -n opa giros
```

### Expondo o pod

Para expor um pode temos que passar a porta:
```
kubectl expose pod nginx --port 80
```

Expondo com o NodePort:
```
kubectl expose pod nginx --port 80 --NodePort
```

Para ver a aplicação do pod em execução pegue o ip de um dos nós do cluster e veja a porta do NodePort (pela qual o pod está sendo exposto).

### Entrando em um pod

```
kubectl exec -ti nginx sh
```

### Obtendo várias saídas (obtendo informação)

```
kubectl get pods,deploy,services,namespaces,nodes
```

Acessando tudo:
```
kubectl get all
```



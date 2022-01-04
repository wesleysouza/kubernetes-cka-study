# Bonde do CKA aula 02

## Questão 1

Levantar algumas informações sobre o cluster:
- Quantos nodes são workers?
- Quantos nodes são master?
- Qual o Pod Network (CNI) que estamos utilizando?
- Qual o CIDR dos pods no segundo worker?
- Qual o DNS que estamos utilizando para o cluster?

Adicionar as informações colhidas no arquivo /opt/cluster_info.txt

### Quantos nodes são workers?

```
kubectl get nodes
```
**Resposta: 2**

### Quantos nodes são master?

```
kubectl get nodes
```

**Resposta: 1**

### Qual o Pod Network (CNI) que estamos utilizando?

```
kubectl get namespaces
```

```
kubectl get pods -n kube-system -o wide | grep weave
```

Outra forma:

Entre no diretório /etc/cni/net.d e veja o arquivo de configuração.

**Resposta: weave-net**

### Qual o CIDR dos pods no segundo worker?

Pegue o IP do nó com o comando abaixo:

```
kubectl describe nodes nome-do-no
```

Entre no nó com o ssh.

```
ssh ip
```

Entre como sudo:

```
sudo su
```

Entre no direório do kubernetes

```
/etc/kubernetes/manifests
```

O manifests contém vários arquivos importantes sobre a criação do cluster.

```
ps -ef
```

Para pegar uma saída mais completa:

```
ps -efwww
```

Observe o ipalloc-range no pod weave-net.

Observe o pod kubelet veja o arquivo /var/lib/kubelet/config.yaml:
- Cluster DNS;

Obtendo as informaçõs pelo jasonpath:

```
kubectl get node -o jsonpath="{range .itens[*]}{.metadata.name} {.spec.PodCIDR}"
```

Melhor opção:

```
kubectl cluster-info dump | grep -i -m 1 clustercidr
```

```
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

Mais uma maneira:

```
kubectl get nodes k8s-worker1 | grep podCIDR
```

Outra maneira:

```
ssh 192.168.86.61
grep -i cidr /etc/kubernetes/manifests/kube-apiserver*
```

**Resposta: "--service-cluster-ip-range=10.96.0.0/12"**

### Qual o DNS que estamos utilizando para o cluster?

```
kubectl get pods -n kube-system -o wide
```

```
kubectl cluster-info dump | grep dns | more
```

**Resposta: "coredns"**

## Questão 2

Precisamos criar um pod com as seguintes características:
- Precisa ter um container rodando a imagem do nginx, com um volume montado no diretorio html do nginx;
- Precisa ter um container rodando busybox e adicionando algum conteúdo ao arquivo /tmp/index.html;
```bash
 => command: ["sh", "-c", "while true; do uname -a >> /temp/index.html; date >> /tmp/index.html; sleep 2; done"]
```
- Precisaos ter outro container rodado o bosybox e executando o seguintes comando:
```bash
 => command: ["sh", "-c", "tail -f /tmp/index.html"]
```

Vendo os logs:
```
kubectl logs -f nome-do-pod nome-container 
```

```
kubectl exec -ti meu-pod -- sh
bash
cd /usr/share/nginx/html
```

```
kubectl exec -ti pod-multi-container -c container-1 -- tail -f /usr/share/nginx/html/index.html
```

**Resposta**:

Criamos o arquivo abaixo definindo o pod com três containers:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-multi-container
spec:
  containers:
  - name: container-1
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
      - name: workdir
        mountPath: /usr/share/nginx/html
    resources:
      limits:
        memory: '1Gi'
        cpu: '800m'
      requests:
        memory: '700Mi'
        cpu: '400m'

  - name: container-2
    image: busybox
    command: ["sh", "-c", "while true; do uname -a >> /temp/index.html; date >> /tmp/index.html; sleep 2; done"]
    volumeMounts:
      - name: workdir
        mountPath: /tmp/

  - name: container-3
    image: busybox
    command: ["sh", "-c", "tail -f /tmp/index.html"]
    volumeMounts:
      - name: workdir
        mountPath: /tmp/

  dnsPolicy: Default
  volumes:
    - name: workdir
      emptyDir: {}
```

Criando o pod:
```bash
kubectl create -f pod.yaml
```

Verificando os logs para verificar se está tudo funcionando:

```
kubectl logs -f nome-do-pod nome-container 
```
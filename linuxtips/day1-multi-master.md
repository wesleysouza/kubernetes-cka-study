# Day 1 - Criando Cluster Multi-Master

## Opções para cluster HA (Highly Available)

Veja a documentação em (Options for Highly Available topology)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/].

Opções:
- Cluster etcd separado;
- Cluster junto do etcd (stacked etcd).

## Load Balancer

Os nodes vão se comunicar por meio do LoadBalancer, é necessário um dns e um IP que aponte para os nós onde estão rodando o controll plane. Além disso, acesso por meio da porta 6443 TCP.

Opções de load balancer:
- (HA proxy)[https://www.haproxy.com/].

### Install HAProxy

Atualize sua máquina:
```shell
sudo apt update && sudo apt upgrade -y
```
Instalando o HAProxy:

```shell
sudo apt install -y haproxy
```

Configure o HAProxy editando o arquivo /etc/haproxy/haproxy.cfg. Não é necessário alterar a configuração básica, basta apenas adicionar o **"frontend"** e o **"backend"**:

- **"frontend"**: o ip que será o VIP (virtual ip), quando alguém tentar conectar com esse IP será imediatamente redirecionado para os masters. 
- **"backend"**: 

Configrando o **"frontend"**:
```
frontend kubernetes
    mode tcp
    bind ip ip-da-maquina:6643 (porta do api server do k8s)
    option tcplog
    defalut_backend k8s-masters (nome do backend que será configurado)
```

Configurando o - **"backend"**:
```
backend k8s-masters
    mode tcp
    balancer roudrobin (algoritmo de balanceamento)
    option tcp-check
    server nome-server ip-do-server check fall 3 rise 2
    ... (coloca aqui o nome dos demais nós)
```
Explicação dos parâmetros:
- **check fall n:** quantidade de verificações realizadas até considerar que o nó caui.
- **rise n** quantidade de verificações realizadas até considerar que o nó ta up.

Salve as modificações e reinicie o serviço com o comando:

```
systemctl restart haproxy
```

Verifique o status:

```
systemctl status haproxy
```

Você também pode verificar os logs:

```
tail -f /var/log/haproxy.log
```

Verifique conexões e portas com o netstat, comando:

```
netstat -atunp
```

### Teste o Load Balancer

```
nc -v LOAD_BALANCER_IP PORT
```

## Configurando o Cluster

Instalando o kubeadm, kubelet e kubectl nos masters:

Primeiros procedimentos:
- update;
- altere o nome das máquinas;
- instale o docker (verifique se o driver d docker é o systemd);

Forma fácil de alterar o nome das máquinas:

```
acho "nome_da_maquina" > /etc/hotname
```

### Instalação do kubeadm, kubelet e kubectl

Fonte: (Installing kubeadm, kubelet and kubectl)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]

Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
Add the Kubernetes apt repository:

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Iniciando o cluster com o Load Balancer

(Stacked control plane and etcd nodes)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/]

```
sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT" --upload-certs
```

Va no HAProxy e edite o arquivo /etc/hosts adicionando os IPs dos nós master. Adicione o conteúdo:

```
ip nome-da-máquina
```

Vc pode também iniciar o cluster usando o nome:

```
sudo kubeadm init --control-plane-endpoint "nome-do-no-master:6443" --upload-certs
```

Obs.: A saída do comando acima vai trazer informações de como adicionar nocos control planes e nós workers.

IMPORTANTE: salve essa saída em algum lugar!

Faça a configuração do kubecontrol.

#### Instalando o pod network

Instale o pod network em um dos nós master.

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Type the following and watch the pods of the control plane components get started:
```
kubectl get pod -n kube-system -w
```

#### Instalando outros nós control plane.

Utilize o token salvo para add os outros nós.

### Add os workers

Coisas a fazer:
- Altere o nome dos workers;
- Add o ip do HAProxy no arquivo /etc/hosts;
- Instale o Docker;
- Instale o Kubernetes (kubeadm, kubelet e kubectl);
- Utilize o token para subir os workers.

# Referências

(Tipos de topologias de K8s multi-master)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/]

(Instalação kubeadm, kubelet e kubectl)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]

(Instalação Kubernetes multi-master)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/]

(HAproxy)[https://www.haproxy.org/]










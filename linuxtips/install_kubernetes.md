# Instalação do Kubernetes no Ubuntu 20.04 LTS

Essa instalação foi realizada em 3 VMs criadas por meio do VirtualBox. A configuração de cada uma das máquinas é detalhada na tabela abaixo:

| host     | user  | ip           | cpus | RAM | configuração de rede |
|----------|-------|--------------|------|-----|----------------------|
| sagan-01 | sagan | 192.168.0.23 | 2    | 2   | placa em modo bridge |
| sagan-02 | sagan | 192.168.0.21 | 2    | 2   | placa em modo bridge |
| sagan-03 | sagan | 192.168.0.22 | 2    | 2   | placa em modo bridge |

## Passo 1: Criar arquivo /etc/modules-load.d/k8s.conf e adicionar o conteúdo abaixo em todos os nós:

```
br_netfilter
ip_vs
ip_vs_rr
ip_vs_sh
ip_vs_wrr
nf_conntrack_ipv4
```

O conteúdo desse arquivo descreve os módulos do kernel GNU/Linux necessários para o pleno funcionamento do Kubernetes.

## Passo 2: Faça update e upgrade do sistema em todosos nós:

```shell
sudo apt update
sudo apt upgrade -y
```

## Passo 3: Instale e configure o Docker em todos os nós

### Instalando o Docker

```shell
curl -fsSL https://get.docker.com | bash
```

#### Adicione o seu usuário no grupo Docker

```shell
sudo usermod -aG docker seu_usuario
```

#### Configuração do Docker Deamon

Cole esse comando no shell:

```shell
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
Restart Docker e ative no boot:

```shell
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

```
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Agora basta reiniciar o Docker.

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Reinicie todos os nós e teste a configuração com o comando:

```
docker info | grep -i cgroup
```

Se a saída foi **Cgroup Driver: systemd**, tudo certo!

## Passo 4: Instale e configure o *kubeadm*.

~~~shell
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl
~~~

### Desativando o swap

Digite o comando abaixo no shell:

~~~shell
sudo swapoff -a
~~~

Comente a linha referente ao swap no arquivo /etc/fstab.

## Passo 5: Inicialize o cluster

Faça download das imagens que vão ser utilizadas no nó master:

~~~shell
sudo kubeadm config images pull
~~~

Execute o comando abaixo no nó master. Caso tudo esteja bem, será apresentada ao término de sua execução o comando que deve ser executado nos demais nós para ingressar no cluster.

~~~shell
sudo kubeadm init
~~~

Exemplo de saída do comando se estiver tudo certo:

~~~shel
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.23:6443 --token d02jsc.tv1mf71vvz8ebu1j \
	--discovery-token-ca-cert-hash sha256:64f7db5fc7dd4b929d70e96608f5574f1195168e9aafae6dd093675a874a0d5a 
~~~

Caso o servidor possua mais de uma interface de rede, você pode verificar se o IP interno do nó do seu cluster corresponde ao IP da interface esperada com o seguinte comando:

~~~shell
kubectl describe node elliot-1 | grep InternalIP
~~~

Caso o Ip não corresponda ao da interface de rede escolhida, você pode ir até o arquivo localizado em /etc/systemd/system/kubelet.service.d/10-kubeadm.conf com o editor da sua preferência, procurar por KUBELET_CONFIG_ARGS e adicionar no final a instrução –node-ip=. O trecho alterado será semelhante abaixo:

~~~shell
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml --node-ip=192.168.99.2"
~~~

Salve o arquivo e execute os comandos abaixo para reiniciar a configuração e consequentemente o kubelet.

~~~shell
sudo systemctl daemon-reload
sudo systemctl restart kubelet
~~~

## Passo 6: Instale o pod network

Carregue os mósulos do kernel necessários com o comando:

~~~shell
sudo modprobe br_netfilter ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack_ipv4 ip_vs
~~~

Baixe o Weave-net, que pode ser instalado com o comando a seguir:

~~~shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
~~~

Para verificar se o pod network foi criado com sucesso, execute o seguinte comando:

~~~shell
kubectl get pods -n kube-system
~~~

O resultado deve ser semelhante ao mostrado a seguir.

~~~shell
NAME                                READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-pfm2c            1/1     Running   0          8d
coredns-66bff467f8-s8pk4            1/1     Running   0          8d
etcd-docker-01                      1/1     Running   0          8d
kube-apiserver-docker-01            1/1     Running   0          8d
kube-controller-manager-docker-01   1/1     Running   0          8d
kube-proxy-mdcgf                    1/1     Running   0          8d
kube-proxy-q9cvf                    1/1     Running   0          8d
kube-proxy-vf8mq                    1/1     Running   0          8d
kube-scheduler-docker-01            1/1     Running   0          8d
weave-net-7dhpf                     2/2     Running   0          8d
weave-net-fvttp                     2/2     Running   0          8d
weave-net-xl7km         
~~~

Se sua saída for semelhante a apresentada acima, sendo possível observar que há três contêineres do Weave-net em execução (provendo a pod network para o nosso cluster), tudo foi configurado corretamente.

# Primeiros Passos

~~~shell
kubctl get nodes
~~~

~~~shell
kubctl describe <resource>
~~~

Exemplo:

~~~shell
kubctl describe nodes nome_do_no
~~~

Taints -> Restrições para os nós

~~~shell
kubectl get pods -n kube-system
~~~

-n é o namespace

Comando para adicionar novos nodes:

~~~shell
kubadm token create --print-join-command
~~~

Describe nos pods

~~~shell
kubctl describe pods -n kube-system
~~~

Ver os Pods em todos os namespaces:

~~~shell
kubectl get pods --all-namespaces
~~~

Ver os namespaces criados

~~~shell
kubectl get namespaces
~~~

Criando namespaces

~~~shell
kubectl create namespace nome_do_namespace
~~~

## Criando o primeiro Pod

~~~shell
kubectl run nginx --image=nginx
~~~

Vendo os Pods:

~~~shell
kubectl get pods
~~~

Vendo os detalhes do Pod criado:

~~~shell
kubectl describe pods nginx
~~~

Vendo o YAML utlizando (gerado) na criação do Pod:

~~~shell
kubectl get pods nginx -o yaml
~~~

## Criando Pods a partir de um template YAML

Redirecione o conteúdo do YAML do Pod para um arquivo novo YAML:

~~~shell
kubectl get pods nginx -o yaml > meu_primeiro_pod.yaml
~~~

Simplificando o arquivo...

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: test
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
  dnsPolicy: ClusterFirst
  restartPolicy: Always
~~~

### Criando e deletando o Pod por meio do template YAML:

Criando o Pod:

~~~shell
kubectl create -f meu_primeiro_pod.yaml
~~~

Deletando o Pod:

~~~shell
kubectl delete -f meu_primeiro_pod.yaml
~~~

### Dry-run

O Dry-run simula a criação de um Pod. Essa simulação pode ser utilizada para gerar um arquivo de template com tudo o que é necessário para criar o Pod desejado.

~~~shell
kubectl run nginx --image=nginx --dry-run=client -o yaml > meu_segundo_pod.yaml
~~~

Criando o segundo Pod:

~~~shell
kubectl create -f meu_segundo_pod.yaml
~~~

## Expondo o Pod fora do cluster

Para expor o Pod, ou seja, torna-lo visível fora do cluster é necessário criar um service. Para criar esse service digite o comando abaixo:

~~~shell
kubectl expose pod nginx
~~~

No entando, isso não sera suficente se você não definiu as portas de acesso a sua aplicação. Caso não tenha feito isso, vá no seu template YAML e defina o campo **ports**:

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx3
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    ports:
    - containerPort: 80
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
~~~

Execute o comando create para o novo template:

~~~shell
kubectl create -f meu_segundo_pod.yaml
~~~

Agora podemos rodar o comando expose:

~~~shell
kubectl expose pod nginx
~~~

Vendo os services:

~~~shell
kubectl get service
~~~

Vendo a descrição do service criado:

~~~shell
kubectl describe services name_service
~~~

Resultado do comando:

~~~shell
Name:              nginx3
Namespace:         default
Labels:            run=nginx
Annotations:       <none>
Selector:          run=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.108.189.39
IPs:               10.108.189.39
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.46.0.1:80,10.46.0.4:80
Session Affinity:  None
Events:            <none>
~~~

Vendo os endpoints (quando vc cria um service, tamb[em está criando um endpoint):

~~~shell
kubectl get endpoints
~~~


Testando o service dentro do cluster IP

~~~shell
curl 10.108.189.39
~~~

### Criando o acesso externo com o NodePort

Editando o service:

~~~shell
kubectl edit service nginx3
~~~

Substitua no arquivo o tipo ClusterIp para NodePort.

Verifique a porta do serviço com o comando:

~~~shell
kubectl get service
~~~

Exemplo do resultado do comando:

~~~
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d4h
nginx3       NodePort    10.108.189.39   <none>        80:31389/TCP   13m
~~~

Agora é só acessar o serviço pelo IP de qualquer máquina do cluster mais a porta do serviço. No caso do nginx3: **IP_da_maquina:31389**.

## Vendo um pouco da documentação do Kubernetes

~~~shell
kubectl explain pods
~~~


~~~shell
kubectl explain services
~~~

Vendo tudo que foi criado:

~~~shell
kubectl gel all
~~~

Outra forma:

~~~shell
kubectl get pods,services,endpoints
~~~

Abreviações:
- pods: po
- service: svc
- deployment: deploy
- namespace: ns

## Apagando tudo

~~~shell
kubectl delete pods nome_pod
~~~

~~~shell
kubectl delete service name_service
~~~

Verifique com o get all se tudo foi apagado.



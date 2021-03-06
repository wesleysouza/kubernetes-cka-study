# Day 4 Notes

Problemas que podem acontecer com dados em containers:
- Perda de dados quando o container quebra. O Kubelet restarta o containr mas com o stado limpo.
- O segundo problema ocorre quando existem arquivos compartilhados entre os containers de um Pod.

A abstração de volume do Kubernetes resolve ambos os problemas.

Um Pod pode usar qualquer vários volumes de tipos diferentes simultaneamente.

- Tipos de volumes efemeros (Ephemeral volume) tem o mesmo ciclo de vida de um pod.
- Volumes persistentes continuam existindo mesmo depois do pod ser deletado.

Quando o pod deixa de existir o kubernetes mata os volumes efemeros, mas não destroí os volumes persistentes.

Obs.: Para qualquer tipo de volume os dados são preservados quando precisamos restasrtar os containers.

Para usar os volumes, especifique em **.spec.volumes** e declare onde vai motar esses volumes nos containers em **.spec.containers[*].volumeMounts**.

Tipos de Volumes
- EmptyDir: Criado junto do pod iniciando sempre vazio.
- Persistent Volume

## Volume EmptyDir

Uma das aplicações é salvar logs de forma temporária.

Definindo o arquivo manifesto do pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-empty-dir
  name: pod-empty-dir
spec:
  containers:
  - image: busybox
    name: pod-empty-dir
    command:
      - sleep
      - "3600"
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
    volumeMounts:
      - mountPath: /giropops
        name: giropops-dir
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: giropops-dir
    emptyDir: {}
```

Criando o pod:

```
kubectl create -f pod-empty-dir.yaml
```

Entrando dentro dos containers do pod:

```
kubectl exec -ti busybox --sh
```

Veja se ta montando:

```
cd giropops
```

### Entrando e analisando o volume

Veja onde o container ta rodando:

```
kubectl get pods -o wide
```

Entre no nó onde ele está rodadando com o ssh, é lá que o volume foi criado.

Entre no diretório:

```
/var/lib/kubelet/pods
```

Use o comando **find** para encontrar os volumes:

```
find . -iname "giropops-dir"
```

Entre no volume.

Remova o pod.

## Persistent Volume (PV)

O PV precisa ser "atachado" no pod, para isso criamos um PersistentVolumeClaim (PVC).

### Configurando o NFS

Criando um NFS:

Instale no master:

```
apt-get install nfs-kernel-server
```

Nos nós workers:

```
apt-get install nfs-common
```

O NFS é uma forma de compartilhar volumes e filesystems.

Criei uma pasta dados (no nó server):

```
mkdir /opt/dados
chmod 1777 /opt/dados/
vim /etc/exports
```

Adicione o conteúdo abaixo no arquivo **/etc/exports**:

```
/opt/dados *(rw,sync,no_root_squash,subtree_check)
```

Digite o comando (no nó master) **sudo exportfs -a** para permitir a montagem do volume em outras máquinas (publicação do compartilhamento). 

Reinicie o serviço com o comando abaixo:

```
sudo systemctl restart nfs-kernel-server
```

Agora podemos vizualizar o compartilhamento por **outros nós** com o comando abaixo:

```
showmount -e ip
```

### Instalando o PV

Crie o arquivo primeiro-pv.yaml e adicione o conteúdo abaixo:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: primeiro-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/dados
    server: 192.168.0.100
    readOnly: false
```

AccessModes volumes:
- ReadWriteMany: Vários nós podem montar esse volume como leitura e escrita.
- ReadWriteOnce: Apenas um nó pode montar esse volume como leitura e escrita.
- ReadOnlyMany: Vários pode montar apenas como leitura.

PersistentVolumeReclaimPolicy: Retain

Esse caso nada acontece com o PV.

Criando o PV:

```
 kubectl create -f primeiro-pv.yaml
```

Vendo se o PV foi criado:

```
kubectl get pv
```

Vendo a descrição do PV:

```
kubectl describe pv primeiro-pv
```

### Criando o PersistentVolumeClain (PVC)

Crie o arquivo primeiro-pvc.yaml e adicione o conteúdo abaixo:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: primeiro-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 800Mi
```

Criando o PVC:

```
kubectl create -f primeiro-pvc.yaml
```

Vendo se o PVC foi criado:

```
kubectl get pvc
```

É possível observar que já o status do PV foi alterado para bound e já é exibido o claim.

### Criando o nosso primeiro Deployment com o nosso PV

Crie o arquivo **nfs-pv.yaml** com o conteúdo abaixo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        volumeMounts:
        - name: nfs-pv
          mountPath: /giropops
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      volumes:
      - name: nfs-pv
        persistentVolumeClaim:
          claimName: primeiro-pvc
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

Criando o Deployment:

```
kubectl create -f nfs-pv.yaml
```

```
kubectl get deployments
```

```
kubectl describe deployment nginx
```

Entrando no pod:

```
kubectl exec -ti nome-do-pod -- sh
bash
```

Criando um arquivo no /giropops:

```
touch TESTE GIROPOPS STRIGUS GIRUS
```

Depois verifique na pasta dados se os arquivos foram criados.

É possível editar o pv:

```
kubectl edit pv primeiro-pv
```

## Cronjobs

Crontab de maneira clusterizada. Por meio dele é possível agendar tarefas que vão ser realizadas em algum momento no cluster.

Crie o arquivo **primeiro-cronjob.yaml** com o conteúdo abaixo:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: giropops-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: giropops-cron
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Bem Vindo ao Descomplicando Kubernetes - LinuxTips VAIIII ;sleep 30
          restartPolicy: OnFailure
```

Formato do cronjob: 

```
minutos hora mes dia-da-semana comando
```

Exemplos:

```
*/1 -> roda a cada um minuto
10 08 * * 1-5
0-59 0-23 1-31 1-12 0-7

0 domingo
1 segunda
2 terça
3 quarta
4 quinta
5 sexta
6 sabado
7 domingo
```

XXXXXXXXXX

Criando o cronjob:

```
kubectl create -f primeiro-cronjob.yaml
```

Vendo os jobs:

```
kubectl get jobs
```

Acompanhando com mais detalhes:

```
kubectl get jobs --watch
```

## Secrets

Passar chave e valor sem expor em um arquivo.

Obs.: É interessante gerenciar secrets pelo vault.

Crie o arquivo secret.txt com o comando abaixo:

```
echo -n "senha boa 123"
```

Criando secret:

```
kubectl create secret generic my-secret --from-file=secret.txt
```

Vendo a secret:

```
kubectl get secrets
```

Vendo os detalhes da secret:

```
kubectl describe secrets my-secret
```

Vendo a descrição por meio de um arquivo yaml:

```
kubectl get secrets nome-da-secret -o yaml
```

No campo **data** é possível observar uma cadeia de caractertes "cifrada". É possível descifras essa cadeia utilizando o comando abaixo:

```
echo cadeia-de-caracteres | base64 --decode
```

Criando o primeiro pod que utiliza uma secret:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-secret
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
      - sleep
      - "3600"
    volumeMounts:
    - mountPath: /tmp/giroposp
      name: my-volume-secret
  volumes:
  - name: my-volume-secret
    secret:
      secretName: my-secret
```

O secret é um driver volume.

Crie o pod:

```
kubectl create -f nome-do-pod.yaml
```

Entre no pod:

```
kubectl exec -ti test-secret -- sh
```

O secret ta montando no /tmp/giropops.

### Crianod secret do nomo literal

```
kubectl create secret generic my-literal-secret --from-literal user=wesley --from-literal password=senha
```

Veja a secret:

```
kubectl get secrets
```

Observe os valores codificados da secret com o comando:

```
kubectl get secret my-literal-secret -o yaml
```

Use o comando abaixo para ver o usuário e a senha:

```
echo "string" | base64 --decode
```

### Uma outra maneira

Crie o arquivo **pod-secret-env.yaml** com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: teste-secret-env
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy-secret-env
    command:
      - sleep
      - "3600"
    env:
    - name: MEU_USERNAME
      valueFrom:
        secretKeyRef:
          name: my-literal-secret
          key: user
    - name: MEU_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-literal-secret
          key: password
```

Criando a chave:

```
kubectl create -f pod-secret-env.yaml
```

Entre dentro do container com o comando abaixo:

```
kubectl exec -ti test-secret-env -- sh
```

Digite o comando **set** e veja as variáveis.

```
kubectl exec teste-secret-env -c busy-secret-env -it -- printenv
```

 Agora podemos utilizar essa chave dentro do contêiner como variável de ambiente, caso alguma aplicação dentro do contêiner precise se conectar ao um banco de dados por exemplo utilizando usuário e senha, basta criar um secret com essas informações e referenciar dentro de um Pod depois é só consumir dentro do Pod como variável de ambiente ou um arquivo texto criando volumes.

 ## ConfigMaps

Forma de adicionar configurações dentro de um container.

```
mdkir frutas
echo amarela > frutas/banana
echo vermelho > frutas/morango
acho verde > frutas/limao
echo "verde e vermelho" > frutas/melancia
echo kiwi > predileta
```

Criando um CongigMap:

```
kubectl create configmap cores-frutas --from-literal uva=roxa --from-file=predileta --from-file=frutas/
```

```
kubectl describe configmap
```

### Criando pod com ConfigMap

Crie o arquivo pod-configmap.yaml com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-configmap
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy-configmap
    command: 
      - sleep
      - "3600"
    env:
    - name: frutas
      valueFrom:
        configMapKeyRef:
          name: cores-frutas
          key: predileta
```

Crie o ConfigMap:

```
kubectl create -f pod-configmap.yaml
```

Entre no container:

```
kubectl exec -ti busy-configmap -- sh
```

```
kubectl delete -f pod-configmap.yaml
```

Edite o arquivo e deixe da forma que ta abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-configmap
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy-configmap
    command: 
      - sleep
      - "3600"
    envFrom:
    - configMapRef:
        name: cores-frutas
```

Entre no container com o comando:

```
kubectl exec -ti busy-configmap -- sh
```

Use o comando **set** para ver as variáveis.


### Criando outro configmap

Crie o arquivo **pod-configmap-file.yaml** com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-configmap-file
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy-configmap
    command: 
      - sleep
      - "3600"
    volumeMounts:
    - name: meu-configmap-vol
      mountPath: /etc/frutas
  volumes:
  - name: meu-configmap-vol
    configMap:
     name: cores-frutas
```

Crie o novo configmap.

```
kubectl create -f pod-configmap-file.yaml
```

```
kubectl exec -ti busybox-configmap-file -- sh
```

Verifique a pasta **/etc/frutas** e veja se se os arquivos e o conteúdo estão lá (eles são links simbólicos).

## InitContainer

Preparar o ambiente para rodar alguma aplicação. Esse container executa alguma ação específica antes do container principal ser executado, preparando o terreno.

Crie o arquivo nginx-initcontainer.yaml com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  initContainers:
  - name: install
    image: busybox
    command: ['wget','-O','/work-dir/index.html','http://linuxtips.io']
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

Crie o initContainer:

```
kubectl create -f nginx-initcontainer.yaml
```

```
kubectl get pods
```

Entre no pod

```
kubectl exec -ti init-demo -- sh
cd /usr/share
ls
cat index.html
```

## RBAC

Criando **serviceaccount** no cluster e atribuíndo permissões a ele.

Criando serviceaccont

```
kubectl create serviceaccount wesley
```

Vendo os servicesaccounts:

```
kubectl get serviceaccounts
```

Vendo as roles:

```
kubeclt cluster role
```

É possível ver as associações de um usuário com determinada role (clusterrolebindings) com o comando abaixo:

```
kubectl get clusterrolebindings.rbac.autorization.ks8.io
```

Vendo as associações da rola **cluster-admin**:

```
kubectl get clusterrolebindings.rbac.autorization.ks8.io cluster-admin
```

Criando um clusterrolebiding:

```
kubectl create clusterrolebinding toskera --serviceaccount=default:wesley --clusterrole=cluster-admin
```

Vendo a clusterrole toskera com mais detalhes:

```
kubectl describe clusterrolebindings.rbac.autorization.ks8.io toskera
```

Vendo com detalhes a conta wesley:

```
kubectl describe serviceacounts wesley
```

## Trabalhando com arquivos yaml

Crie o arquivo admin-user.yaml com o conteúdo abaixo:

```yaml
apiVesion: v1
kind: ServiceAccount
metadate:
  name: admin-user
  namespace: kube-system
```

Crie o service account:

```
kubectl create -f admin-user.yaml
```

Veja se o service account foi criado:

```
kubectl get serviceaccounts -n kube-system
```

Agora vamos criar o **cluserrolebinding**. Crie o arquivo **admin-cluster-role-binding.yaml** e adicione o conteúdo abaixo: 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

Crie o objeto:

```
kubectl create -f admin-cluster-role-binding.yaml
```

Pegue com o get:

```
kubectl get clusterrolebindings.rbac.autorization.ks8.io
```

## Helm

O Helm é o gerenciador de pacotes do Kubernetes. Os pacotes gerenciados pelo Helm, são chamados de charts, que basicamente são formados por um conjunto de manifestos Kubernetes no formato YAML e alguns templates que ajudam a manter variáveis dinâmicas de acordo com o ambiente. O Helm ajuda você a definir, instalar e atualizar até o aplicativo Kubernetes mais complexo.

Obs.: As anotações do Helm vão ficar no arquivo **day4_aulao**.

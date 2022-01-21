# Day4 Aulão

Conteúdo:
- StorageClass
- Volumes
- CronJobs
- Secrets
- ConfigMaps
- InitContainers
- RBAC
- Helm

## Volumes

Forma de deixar os seus dados persistidos.

### EmptyDir

Volume do tipo vazio e sem persistência.

Crie o arquivo **pod-emptydir.yaml** com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    name: busy
    command:
      - sleep
      - "3600"
    volumeMounts:
    - mountPath: /giropops
      name: giropops-dir
  volumes:
  - name: giropops-dir
    emptyDir: {}
```

Crie o pod com o comando:

```
kubectl create -f pod-emptydir.yaml
```

Entre no pod e veja se o diretório relacionado ao volume está lá:

```
kubectl exec -ti busybox -- sh
cd giropops/
touch FILE
```

Veja onde o pod está em execução:

```
kubectl get pod -o wide
```

Entre no nó onde esse pod está rodando e procure pelo volume com o comando:

```
find /var/lib/kubelet/pods/ -iname giropops-dir
```

Obs.: **O diretório /var/lib/kubelet/pods/ é onde está aramzeada as informações dos pods**.

Entre na pasta com o subcaminho volumes e veja o arquivo que foi criado.

## Persisten Volume (PV) e Persistent Volume Claim (PVC)

Criando o nosso primeiro PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadate:
  name: primeiro-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/giropops
    server: 192.168.0.100
    readOnly: false
```

Crie o PV. Veja se o PV subiu:

```
kubectl get pv
```

### Criando o PVC

Crie o arquivo **primeiro-pvc.yaml** com o conteúdo abauxo:

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

Crie o PVC:

```
kubectl create -f primeiro-pvc.yaml
```

Verifique se o PVC "grudou" no seu PV:

```
kubectl get pv
kubectl get pvc
```

Veja mais detalhes do seu PVC:

```
kubectl describe pvc primeiro-pvc
```

### Criando um Deployment

Crei um Deployment com as definições abaixo:

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

Crie o Deployment:

```
kubectl create -f nfs-pv.yaml
```

Veja os detalhes do Deployment:

```
kubectl describe deployment nome-deployment
```

Entre em um dos pods do Deployment com o comando abaixo:

```
kubectl exec -ti id-do-container sh
```

Entre no diretório do PV, o **/giropops** e verifique se ta ok. 

### Associando PV a um PVC

Para fazer isso vamos precisar usar uma label no campo **selector**.

Fica de atividade.

## Cronjobs

Programar a execução de tarefas.

Crie o arquivo **meu-primeiro-cron-job.yaml** com o conteúdo abaixo:

```yaml
apiVersion: batch/v1beta1
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

Crie o cronjob e verifique-o com o describe.

Acompanhando os Cronjobs:

```
kubectl get jobs.batch --watch
```

## Initcontainer

O Initcontainer prepara o terreno para o container principal.


Crie um Initcontainer pelo arquivo nginx-initcontainer.yaml com o conteúdo abaixo:


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

```

```






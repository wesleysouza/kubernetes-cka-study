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

Minuto 49:31

```

```

```

```

```

```

```

```

```


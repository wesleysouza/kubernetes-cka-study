# Bonde do CKA - Aula 08

## Questão 1

Precisamos subir um pod. Porém esse pod somente poderá ficar disponível quando um determinado service estiver no ar.

O serviço deverá ser um simples nginx.

O pod, nós teremos mais detalhes durante a resolução.

```
kubectl run giropops --image nginx --port 80 --dry-run=client -o yaml > pod-que-espera-o-service.yaml 
```

Altere o arquivo de definição criado e deixe da masma maneira do apresentado abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-que-espera-o-service
  name: pod-que-espera-o-service
spec:
  containers:
  - image: nginx
    name: pod-que-espera-o-service
    ports:
    - containerPort: 80
    resources: {}
    livenessProbe:
      exec:
        command:
        - 'true'
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - 'wget -T4 -O- http://my-nginx:80'
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Probes:
- liveness: ta vivo?
- readiness: ta pronto?

Crie o pod e verifique o **livenessProbe** e o **readinessProbe** com o describe:

```
kubectl describe pods nome-do-pod
```

Observe que o pod não está rodando, ele ainda está esperando o serviço para subir.

```
NAME                       READY   STATUS    RESTARTS   AGE
pod-que-espera-o-service   0/1     Running   0          42s
```

Veja os detalhes do pod com o comando abaixo:

```
kubectl describe pod pod-que-espera-o-service
```

**Copie o arquivo manifesto do pod em um pod novo**. Modifique-o e deixe com o conteúdo apresentado abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-nginx
  name: my-nginx
spec:
  containers:
  - image: nginx
    name: my-nginx
    ports:
    - containerPort: 80
    resources: {}i
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Crie esse pod. E verifique se ele está rodando.

**Agora vamos criar o serviço**

```
kubectl expose pod my-nginx
```

Ainda não subiu!

**Não tem o wget no container, daí da erro!**

Resolução: basta instalar o wget no pod!

```

```

```

```

```

```

```

```

```


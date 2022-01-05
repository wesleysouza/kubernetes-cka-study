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



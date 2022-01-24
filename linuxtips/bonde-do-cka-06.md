# Bonde do CKA 06

## Questão 1

O nosso gerente observou no dashborad Lens que um dos nosos nodes não está bem. Temos algum problema com o nosso cluster e precisamos resolver agora.

Verifique os nós:

```
kubectl get nodes
```

Um dos nós está **NotReady**.

Passo 1: Verifique o kubelet

```
systemctl status kubelet
```

Ele está reclamando do arquivo /usr/local/bin/kubelet. Esse arquivo não existe. 

Utilize o comando whereis para encontrar o caminho do binário do kubelet:

```
whereis kubelet
```

Logo, o caminho está errado!

Para resolver vamos concertar o caminho do kubelet no arquivo **10-kubeadm.conf**:

```
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Altere o campo ExecStart e execute os comandos:

```
systemctl deamon-reload
systemctl restart kubelet
systemctl status kubelet
```

Outro comando para verificar se o kubelet está rodando:

```
ps -ef | grep kubelet
```

### Resposta

```bash
kubectl get nodes
ssh node_com_problema
ps -ef | grep kubelet
docker ps
systemctl -u kubelet
whereis kubelet
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf #Arrumar o caminho do binário do kubelet
systemctl deamin-reload
systemctl restart kubelet
systemctl status kubelet
journalctl -u kubelet
```

Outra maneira:

```bash
systemctl edit --full kubelet #Aqui também podemos alterar o caminho.
```

## Questão 2

Temos um secret com o nome e senha de usuário que nosa aplicação vai utilizar, precisamos colocar esse secret em um pod. Esse secret deve se tornar uma variável de ambiente dentro do container.

Criando um secret:

```
kubectl create secret generic --help
```

```
kubectl create secret generic credentials --from-literal user=wesley --from-literal password=giros --dry-run=client -o yaml
```

O usuário e senha do arquivo estão codificados, para decodificar utilize o comando:

```
acho "string" | base64 -d
```

Olhando as secrets

```
kubectl get secrets
```

Criando o pod:

```
kubectl run nome-do-pod --image nginx --dry-run=client -o yaml > pod-com-secret.yaml
```

Entre em spec do container e no campo resource faça as seguintes modificações:

```yaml
spec:
...
  containers:
  ...
    env:
    - name: MEU_USER
      valueFrom:
        secretKeyRef:
          name: credentials
          key: user
    - name: MEU_PASSWORD
      valueFrom:
        secretKeyRef:
          name: giropops
          key: password
    #Montando do arquivo.
    volumeMounts:
    - name: credentials
      mounthPath: /opt/giropops
    ...
    volumes: 
    - name: credentials
      secret:
        secretName: credentials
```

Crie o pod:

```
kubectl create -f nome-do-arquivo.yaml
```

Entre no pod:

```bash
kubectl exec -ti nome-pod -- bash
env
cd /opt/giropops/ #Nessa pasta está os arquivos
```

### Resposta

```
kubectl create secret generic credentials --from-literal user=wesley --from-literal password=giros --dry-run=client -o yaml
```

```
kubectl run nome-do-pod --image nginx --dry-run=client -o yaml > pod-com-secret.yaml
```

Alterações necessárias no pod-secret:

```yaml
spec:
...
  containers:
  ...
    env:
    - name: MEU_USER
      valueFrom:
        secretKeyRef:
          name: credentials
          key: user
    - name: MEU_PASSWORD
      valueFrom:
        secretKeyRef:
          name: giropops
          key: password
    #Montando do arquivo.
    volumeMounts:
    - name: credentials
      mounthPath: /opt/giropops
    ...
    volumes: 
    - name: credentials
      secret:
        secretName: credentials
```

Crie o pod e show!

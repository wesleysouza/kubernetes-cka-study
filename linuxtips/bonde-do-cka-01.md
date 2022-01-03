# Bonde do CKA aula 01

## Questão 1

Crie um pod utilizando a imagem do nginx 1.18.0, com o nome giropops no namespace strigus.

Obs.: Geralmente o namespace já vai estar lá. Caso você queira verificar use o comando:

```
kubectl get namespaces
```

Criando o namespace:

```
kubectl create namespace strigus
```

Criando um deployment com esse pod com o dry-run:

```
kubectl create deployment giropops --image nginx:1.18.0 --port 80 --namespace strigus --dry-run=client -o yaml > q1-deployment.yaml
```

Criando o pod através da documentação:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops
  namespace: strigus
spec:
  containers:
  - name: web
    image: nginx:1.18.0
    ports:
    - containerPort: 80
```

Crie o pod:

```bash
kubectl create -f q1.yaml
```

Verifique se ele foi criado:

```
kubectl get pods -n strigus
```

### Resposta: Para resolver é possível fazer de duas formas.

A primeira utilizando a linha de comando:

```bash
kubectl run nome-do-container --image nginx:1.18.0 --port 80 --namespace strigus
```

A segunda, e a mair recomendada é pelo arquivo, pois você terá mais controle da criação. 

```bash
kubectl run nome-do-container --image nginx:1.18.0 --port 80 --namespace strigus --dry-run=clisnt -o yaml > nome-do-pod-yaml
```

```
kubectl create -f pod.yaml
```

## Questão 2

Aumentar a quantidade de réplicas do deployment girus, que está utilizando a imagem do nginx 1.18.0, para 3 réplicas. O Deployment está no namespace strigus.

Crie o deployment:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops
  namespace: strigus
spec:
  containers:
  - name: web
    image: nginx:1.18.0
    ports:
    - containerPort: 80
```

Outra opção de criação é pelo **dry-run**:

```
kubectl create deployment q2-deployment --image=nginx:1.18.0 -n strigus --dry-run=client -o yaml > q2-deployment.yaml
```

Criando o deployment:

```
kubectl create -f q2-deployment.yaml
```

Verificando os pods:

```
kubectl get pods -o wide
```

Obs.: Na prova o deployment já existe.

Verifique o deployment

```bash
kubectl create -f deployment.yaml -n strigus
```

### Resposta

Aumentando a quantidade de réplicas com o scale:

```bash
kubectl scale deployment -n strigus q2-deployment --replicas 3
```

Opção 2, alterar no arquivo no campo réplicas para o valor desejado e depois aplicar a modificação com o comando abaixo:

```bash
kubectl apply -f q2-deployment.yaml
```

## Questão 3

Precisamos atualizar a versão do nginx do pod giropos. Ele está na versão 1.18.0 e precisamos atualizar para a versão 1.21.1.

### Resposta

Verifique se o pod ta rodando com o comando: 

```
kubectl get pods -n strigus
```

Verifique a versão com o describe:

```
kubectl describe pod -n strigus giropops
```

A primeira opção para resolver é com o **kubectl edit**, com ela apenas precisamos modificar o arquivo que será aberto e salvar.

```
kubectl edit pod -n strigus giropops
```

Outra opção:

```
kubectl set image pod -n strigus web
=nginx:1.21.1
```


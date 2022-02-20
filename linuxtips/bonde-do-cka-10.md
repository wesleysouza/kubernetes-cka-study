# Bonde do CKA - Aula 10

## Questão 1

Nosso gerente precisa reportar para o nosso diretor, quais as namespaces que nós temos hoje em produção.
Salvar a lista de namespaces no arquivo /tmp/giropops-k8s.txt

Por padrão mesmo em namespaces diferentes os recursos conseguem se comunicar, para evitar isso é necessário criar network polices.

```
kubectl get ns --no-headers -o custom-columns=":metadata.name"> /tmp/giropops-k8s.txt
cat /tmp/giropops-k8s.txt
```

Saída:

```
aula02
catota
default
ingress
kube-node-lease
kube-public
kube-system
strigus
test
```

Para ver as opções:

```
kubectl get -o --help
```

## Questão 2

Precisamos criar um pod chamado web e utilizando a imagem do Nginx na versão 1.21.4. O pod deverá ser criado no namespace web-1 e o container deverá ser chamar meu-web-container.

O nosso gerente pediu para que seja criado um script que retorne o status desse pod específico. O nome do script é /tmp/script-do-gerente-toskao.sh

Criando o namespace:

```
kubectl create ns web-1
```

Criando o pod com o dry-run:

```
kubectl run web --image=nginx:1.21.4 --port=80 --namespace=web-1 --dry-run=client -o yaml > meu-web-container.yaml
```

Arquivo manifesto:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
  namespace: web-1
spec:
  containers:
  - image: nginx:1.21.4
    name: meu-container-web
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Crie o pod. Agora vamos criar um script para retornar um status:

```bash
kubectl get pods -n web-1 --no-headers -o custom-columns=":metadata.name,:status.phase"
```

Esse **:status.phase** é pego da estrutura arquivo manifesto em execução, é possível ver esse arquivo com o comando:

```
kubectl get pods -n web-1 -o yaml
```

Edite o arquivo de script da maneira apresentada abaixo:

```bash
kubectl get pods -n web-1 --no-headers -o custom-columns=":metadata.name,:status.phase"
```

Dê para o arquivo poder de execução:

```
chmod +x
```

Agora podemos executar o arquivo.

## Questão 3

Criamos o pod do Nginx, parabéns!
### TASK-1
Portanto, agora precisamos mudar a ver~soa do Nginx para a versão 1.18.0, pois o nosso gerente viu um artigo no Medium e disse que agora temos que usar essa versão e ponto.
### TASK-2
Precisamos criar um deployment no lugar do nosso pod do Nginx.
### TASK-3
Precisamos utiliar o Nginx com a imagem do Alpine, porque o gerente leu outro artigo no medium
### TASK-4
Precisamos realizar o rollback do nosso deployment web

### TASK-1: Fazendo o update

Editando versão:
```
k edit pods -n web-1
```

#### Fazendo roolback:

Verificando o histórico

```
k rollout history -n web-1
```

Não é possível fazer rollback em pod.

### TASK-2: Criando Deployment

Criando o deployment:

```bash
#Gerando o arquivo manifesto
kubectl create deployment deployment-tesk --image=nginx:1.21.4 --port=80 --dry-run=client -o yaml > deployment-test.yaml
#Comando para criação
kubectl create -f deployment-test.yaml
```

### TASK-3: Mudando a imagem

Editando o deployment:

```
kubectl edit deployments.apps -n web-1 web
```

Agora é necessário mudar o nome da imagem para **nginx:1.20.2-alpine**.

### TASK-4: Rollback

Ajuda com o rollout:

```
kubectl rollout --help
```

Vendo o histórico:

```
kubectl rollout history deployment -n web-1 deployment-tesk
```

Saída:

```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Vendo as revisões:

```
kubectl rollout history deployment -n web-1 deployment-tesk --revision=1
```

Voltando para a revisão 1:

```
kubectl rollout undo deployment -n web-1 deployment-tesk --to-revision=1
```

EXTRA: Adicionando o chance cause usando a opção **--record**:

```
kubectl history deployment -n web-1 deployment-tesk --replicas 4 --record
```
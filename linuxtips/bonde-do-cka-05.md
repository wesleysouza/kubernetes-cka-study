# Bonde do CKA 05

## Questão 01

Precisamos subir um container em um node master. Esse container tem que estar rodando a imagem do nginx, o nome do pod é pod-web e o container é container-web. Ele deve ser criado no namespace catota.

A primeira coisa que temos que fazer é criar o namespace. Mas antes verifique se ele existe:

```
kubectl get ns
```

Se ele ainda não existe crie:

```
kubectl create ns catota
```

Criando o pod com o dry-run:

```
kubectl run pod-web --image=nginx --dry-run=client --namespace=catota -o yaml > pod-web.yaml
```

Agora precisamos subir no nó master, mas tem o Taint que impede isso (No Schedule).

Obs.: Não é recomendável subir containers no master!

Documentação: Taint and Tolerations

Verifique se os nós workers tem algum taint ativado e pods rodando.

Obs.: É interessante que já exista um deployment rodando na máquina.

Adicionando uma Taint:

```
kubectl node seul-cool-05 key1=value1:NoExecute
```

Removendo o Taint:

```
kubectl node seul-cool-05 key1=:NoExecute-
```

Fazendo o balanceamento:

```
kubectl rollout status deployment nome-deployment
kubectl rollout restart deployment nome-deployment
```

### Hora de fazer rodar no master

O **Tolaration** permite que a gente ignore algum Taint.

```
kubectl describe node nome-do-no | grep -i taint
```

Definindo o **Toleration** no pod pra que ele rode em nós master:

```yaml
spec:
...
  tolerations:
    - effect: NoSchedule
      operator: Equal
      key: node-role.kubernetes.io/master
```

Agora vamos selecionar em que nó ele deve rodar utilizando o **NodeSelector**:

```yaml
spec:
...
  nodeSelector:
      node-role.kubernetes.io/master: ""
```

### Resposta

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-web
  name: pod-web
  namespace: catota
spec:
  containers:
  - image: nginx
    name: container-web
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:
  - effect: NoSchedule
    operator: Equal
    key: node-role.kubernetes.io/master
  nodeSelector:
    node-role.kubernetes.io/master: ""
status: {}
```

## Questão 2

Precismoa de algumas informações do nosso cluster e dos pods que lá estão. Portanto, precisamos do seguinte:
- Adicione todos os pods do cluster por ordem de criação, dentro do arquivo /tmp/pods.txt
- Remova o pod do weave, verifique os eventos e os adicione os eventos no arquivo /tmp/eventos.txt
- Liste todos os pods que estão em execução em um determinado nó e os adicione no arquivo /tmp/pods/nome-do-no.txt

### Ordem de criação

Existe uma opção de sort que possibilita uma saída ordenada dos pods usando como chave de ordenação a data da criação, vamos procura-lá utilizando o help:

```
kubectl get pods --help
```

Utilziando a opção **--sort-by=**:

```
kubectl get pods --sort-by=.metadata.creationTimestamp -A
```

É possível deixar a saída mais organizada utilizando o option **-o**. Por exemplo, pegando apenas o nome:

```
kubectl get pods --sort-by=.metadata.creationTimestamp -A -o name
```

**IMPORTANTE**: Verifique a estrutura do arquivo de definição para selecionar o atributo corretamente.

**Dica:** Se vc digitar kubectl get pods -o "qualquer coisa" ele vai dar erro e mostrar as opções adequadas.

#### Resposta:

```
kubectl get pods --sort-by=.metadata.creationTimestamp -A -o name > /tmp/pods.txt
```

### Removendo o weave

Verifique os pods:

```
kubectl get pods -n kube-system
```

Pegando os eventos de todos os namespaces:

```
kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp
```

Matando o pod:

```
kubectl delete pods -n kube-system weave-pod-name
```

Observe os eventosno intervalo iniciando onde ele foi deletado e finalizando quando ele é substituído.

Obs.: Existe a opção --no-headers que pode ser utilizada para melhorar a saída para ser tulizada em automações. Exemplo:

```
kubectl get pods --no-headers
```

### Lista de pods

```
kubectl pods --all-namespaces -o wide --field-selector=
```

Para ver o campo, utilize o comando para ver a descrição do objeto em yaml:

```
kubectl get pods nome-do-pod -o yaml
```

Utilizando o atributo:

```
kubectl get pods --all-namespaces --field-selector spec.nodeName=NomeDoNo
```

#### Resposta

```
kubectl get pods --all-namespaces --field-selector spec.nodeName=NomeDoNo -o name > /tmp/pods-nome-do-no.txt
```
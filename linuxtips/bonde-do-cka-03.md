# Bonde do CKA aula 02

Assuntos de hoje:
- Pod Estático
- Certificados

## Questão 1

Criar um pod estático utilizando a imagem do nginx.

- **Pod Estático**: Pod diretamente criado e gerenciado pelo kubelet. É utilizando quando precisamos que um pod rode em um nó específico.
- **--pod-manifest-path**: Caminho do pod estático.

Criando com o **dry-run**:

```
kubectl run pod-estatico --image nginx -o yaml --dry-run=client > pod-estatico.yaml
```

Passe com o scp o arquivo para o nó master:

```
scp nome-do-arquivo.yaml ip:/tmp
```

Entre no nó master com o ssh.

```
ssh ip
```

entre como sudo.

```
cd /etc/kubernetes
ls
vim kubelet.conf
cd manifests 
```

O arquivos de manifests definem as coisas que devem subir no nó.

Copie o arquivo para dentro de manifests:

```
cp /tmp/nome-do-pod-estatico .
```

Agora verifique se foi criado com o comando abaixo:

```
kubectl get pods
```

**Obs.: Para deletar o pod estático basta remover ele do diretório /etc/kubernetes/manifests**.

### Resposta

Explicação: Para criar um pod estatico, você precisa adicionar o manifesto de criação do pod desejado, dentro do diretório /etc/kubernetes/manifests.

```
cd /etc/kubernetes/manifests
vim meu-pod-estatico.yaml
```

## Questão 2

O nosso gerente está assustdo, pois conversando com o gerente de outra empresa, ficou sabendo que aconteceu uma indisponibilidade no ambinete kubernetes devido a certificados expirados. Ele uqer que tenhamos a certeza que o nosso cluster não corre esse perigo. Portanto, adicione no arquivo /tmp/meus-certificados.txt todas as datas de expiraçaõ.

Obs.: Os certificados se renovam automáticamente.

Verifique os nós:

```
kubectl get nodes
```

Entre em um dos nós master e entre no diretório abaixo:

```
/etc/kubernetes/pki 
ls
```

Esse diretório mostra os certificados.

```
openssl x509 -noout -text nome-do-certificado | grep -i "Not Af -noout -subject -enddate -in {}ter"
```

Outra maneira:

```
openssl x509 -noout -text nome-do-certificado | grep -i -A3 Validity 
```

Maneira mais direta:

```
find /etc/kubernetes/pki/ -name "apiserver*crt" -exec openssl x509 -noout -subject -enddate -in {} \; >> /tmp/meus-certificados.txt
```

### Resposta

Os certificados, por padrão, ficam no diretório /etc/kubrnetes/pki/. Para que você possa verificar a data de expiraçaõ, você pode utilizar o openssl conforma abaixo:

```
cd /etc/kubernetes/pki
openssl x509 -noout -text -in apiserver.crt | grep -i "not after"
```

Lembrar de adicionar a data de expiração no arquivo solicitado.

Podemos também utilizar a forma mais direta.

```
find /etc/kubernetes/pki/ -name "apiserver*crt" -exec openssl x509 -noout -subject -enddate -in {} \; >> /tmp/meus-certificados.txt
```

### Maneira ainda mais fácil: Certificate Management with Kubeadm

O comando abaixo já tras a resposta pronta:

```
kubeadm certs check-expiration > /tmp/meus-certificados.txt
```

#### Renovando certificados:

Renovando todos os certificados:

```
kubeadm certs renew all
```

Para mais opções consulte o help:

```
kubeadm certs renew --help
```


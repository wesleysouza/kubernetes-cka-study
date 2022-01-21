# Bonde do CKA - Aula 04

Assunto de hoje: **etcd**

## Questão 01

O nosso gerente solicitou que seja feita agora, um backup/snapshot do nosso ETCD. Ele ficou muito assustado em saber que se perdermos o ETCD, perderemos o nosso cluster, e consequentemente, a nossa tranquilidade! Portanto, bora fazer esse snapshot imediatamente!

nomeDoServiço-nomeDoNó -> pod estático, a localização dos pods estáticos está em **/etc/kubernetes/manifests/**. O arquivo é o **etcd.yaml**.

Para realizar o backup é necessário verificar os certificados. É possíver ver os certicados com o comando:

```
grep etcd kube-apiserver.yaml
```

Instale o pacote etcd-client:

```
sudo apt-install etcd-client
```

Use o comando abaixo para ver os certificados que precisam ir no backup:

```
etcdctl --help
```

Na saída é descrito os certificados que devem ser salvos junto do backup:

- --cert-file value          
- --key-file value       
- --ca-file value

Veja novamente os certificados:

```
grep etcd kube-apiserver.yaml
```

Rode o comando:

```
ETCDCTL_API=3 etcdctl snapshot save o-backup-do-gerente.db --key /etc/kubernetes/pki/apiserver-etcd-client.key --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --cacert /etc/kubernetes/pki/etcd/ca.crt
```

Veja se ta tudo ok e analise o arquivo:

```
ls -lha o-backup-do-gerente.db
```

Para ter ajuda use o comando:

```
ETCDCTL_API=3 etcdctl snapshot --help
```

### Resposta:

```bash
ssh node-master # Um dos nodes onde o ETCD está em execução
cd /etc/kubernetes/manifests
cat etdc.yaml
grep etcd kube-apiserver.yaml
# Com essas informações, já podemos criar o nosso snapshot!
ETCDCTL_API=3 etcdctl snapshot save o-backup-do-gerente.db --key /etc/kubernetes/pki/apiserver-etcd-client.key --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --cacert /etc/kubernetes/pki/etcd/ca.crt
```

**Obs.: Talvez seja necessário transferir esse arquivo para um local seguro.**


1:10:23

```

```

```

```


```

```



```

```


```

```


```

```


```

```


```

```


```

```


```

```
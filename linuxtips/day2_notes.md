# Day 2

## Services

É uma forma de pegar o deployment e permitir que ele seja acessado de fora do cluster. 

Afirmações importantes sobre a arquitetura:
- Controler de um POD é o ReplicSet;
- Controller do ReplicaSet é o Deployment;
- Se um POD morrer o ReplicaSet é o controller responsável por subir outro.
- Atualização de um ReplicaSet (atualização de uma imagem, alteração do deployment), derruba o ReplicaSet atual e sobe um novo. O antigo permacene para situações de roolback.
- Namespace: separação do cluster por grades. No namespace é possível aplicar cotas.
- KubeAPIServer: responsável por toda a comunicação.
- KubeProxy: responsável por gerenciar a rede dos containers, liberação de portas e etc.

CNI: O Kubernetes não vai fazer a sua rede funcionar por completo... Toda a comunicação de rede entre os Pods é por IP, não usa NAT. O CNI é uma biblioteca para que vc faça o eu plugin para que a rede funcione (rede do container). O wevenet é um desses plugins.

O WaveNet atua na camada 2 e usa o VXLAN.

```

```
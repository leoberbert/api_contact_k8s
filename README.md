### API RestFul com Python, Flask e MongoDB no K8S(kubernetes) ###

Para executar este projeto, é necessário que você já tenha um cluster k8s + NFS configurado, para isso basta seguir o tutorial "https://github.com/leoberbert/cluster-dev-k8s"

Este projeto, trata-se de uma API em Python utilizando flask que se conecta a um banco de dados mong para cadastro de contatos.

Após conectar na máquina master:
``` 
vagrant ssh master
```
Execute os comandos abaixo:

```
git clone https://github.com/leoberbert/api_contact_k8s.git

cd api_contact_k8s

```
Primeiramente criaremos o volume persistente que aponta para a máquina de storage, utilizando o comando abaixo:

```
vagrant@master:~/api_contact_k8s$ kubectl create -f mongo-db-pv.yaml
persistentvolume/nfs-v1 created
persistentvolume/nfs-v2 created
persistentvolume/nfs-v3 created

```
O arquivo mongo-db-pv.yaml, possui todos os volumes persistentes disponíveis em nossa máquina storage.

Agora faremos a configuração do "PersistentVolumeClaim", note que o name do metadata é onde será referenciado qual volumete vamos utilizar, dentro do arquivo mongo-db-pvc.yaml.

```
vagrant@master:~/api_contact_k8s$ kubectl create -f mongo-db-pvc1.yaml
persistentvolumeclaim/nfs-pvc1 created

```
Em seguida vamos fazer o deploy da imagem e do serviço tanto do banco de dados do mongo e da nossa api em python. para isso utilizaremos os seguintes comandos:

```
vagrant@master:~/api_contact_k8s$ kubectl create -f mongo-db-deployment.yaml
service/mongodb created
deployment.apps/mongodb created

vagrant@master:~/api_contact_k8s$ kubectl create -f api_contact.yaml
service/api created
deployment.apps/api created

```
Em seguidas vamos verificar se os pods e os servições foram criado:

```
vagrant@master:~/api_contact_k8s$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
api          ClusterIP   10.107.111.139   <none>        80/TCP      60s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     42m
mongodb      ClusterIP   10.98.12.1       <none>        27017/TCP   86s

vagrant@master:~/api_contact_k8s$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
api-965779fdc-vwh75        1/1     Running   0          116s
mongodb-76bc897b5c-t2vhl   1/1     Running   0          2m22s

```
Obs: Caso o comando acima retorne "ContainerCreating" na coluna status, favor aguardar até que a mesma esteja em "Running".

Em seguida vamos verificar se nossa api está funcionando e realizando o cadastro de contatos, para isso vamos fazer os passos abaixo:

```
cd files/

vagrant@master:~/api_contact_k8s/files$ curl -H "Content-Type: application/json" --data @add_contact.json http://10.107.111.139/add_contact

{"message":"SUCCESS"}

```
Note que o cadastro foi realizado com sucesso e agora vamos consultar o registro cadastrado:

```
curl -XGET -H "Content-Type: application/json" --data @get_contact.json http://10.107.111.139/get_contact

[_id": {"$oid": "5ee66aa56c7b5c73002fba42"}, "name": "Joao Grilo", "contact": "11970801010"}]

```
Notem que o IP "10.107.111.139" trata-se do IP do serviço que foi criado para a api em questão, feito anteriormente, então para realizar os testes, utilize o ip que for gerado no momento da criação do serviço da api.

Espero que com esse tutorial, você tenha conseguido entender como funciona um deploy dentro do K8S(kubernetes).

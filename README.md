# java-kubernetes
Projeto de estudos de Java com Docker e Kubernetes. Nesse projeto é possivel fazer deploy no cluster e configurar nossa a aplicação a fim de fazer debug enquanto ela está rodando no Kubernetes.

O projeto teve como base o: https://github.com/sandrogiacom/java-kubernetes


## Base do app:

### Requirementos:
Java 15
Docker
Make
kubectl
minikube

Ajuda para instalar as ferramentas:
https://github.com/sandrogiacom/k8s

### Contruindo e executando a aplicação:

Spring boot and mysql database running on docker

**Clone o repósitório**
```bash
git clone https://github.com/sandrogiacom/java-kubernetes.git
```

**Contruindo a aplicação**
```bash
cd java-kubernetes
mvn clean install
```

**Iniciando o banco de dados**
```bash
make run-db
```

**Executando a aplicação**
```bash
java --enable-preview -jar target/java-kubernetes.jar
```

**Verificando**

http://localhost:8080/app/users

http://localhost:8080/app/hello

## Segunda Parte - Aplicação no docker:

Criação a partir do Dockerfile:

```yaml
FROM openjdk:15-alpine
RUN mkdir /usr/myapp
COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java --enable-preview $JAVA_OPTS -jar app.jar" ]
```

**Construindo a aplicaçãop e a imagem docker*

```bash
make build
```

Criando e executando o banco de dados
```bash
make run-db
```

Criando e executando a aplicação
```bash
make run-app
```

**Verificando**

http://localhost:8080/app/users

http://localhost:8080/app/hello

Para tudo:

`
docker stop mysql57 myapp
`

## Part three - app on Kubernetes:

Prepare

### Iniciando o minikube
`
make k-setup
`

### Verificando o IP

`
minikube -p dev.to ip
`

### Minikube dashboard

`
minikube -p dev.to dashboard
`

### Deploy database

Criando o deploy do mysql

`
make k-deploy-db
`

`
kubectl get pods -n dev-to
`

Ou

`
watch k get pods -n dev-to
`


`
kubectl logs -n dev-to -f <pod_name>
`

`
kubectl port-forward -n dev-to <pod_name> 3306:3306
`

## Contruindo o deploy da aplicação

build app

`
make k-build-app
` 

Criando a imagem docker dentro do minikube

`
make k-build-image
`

Ou

`
make k-cache-image
`  

Criando o deploy e serviço da aplicação

`
make k-deploy-app
` 

**Verificando**

`
kubectl get services -n dev-to
`

Para acessar a aplicação:

`
minikube -p dev.to service -n dev-to myapp --url
`

Ex:

http://172.17.0.3:32594/app/users
http://172.17.0.3:32594/app/hello

## verificando os pods

`
kubectl get pods -n dev-to
`

`
kubectl -n dev-to logs myapp-6ccb69fcbc-rqkpx
`

##Mapeando dev.local

Retornando IP minikube
`
minikube -p dev.to ip
` 

Edição `hosts` 

`
sudo vim /etc/hosts
`

Replicas
`
kubectl get rs -n dev-to
`

Listando e deletando pods
`
kubectl get pods -n dev-to
`

`
kubectl delete pod -n dev-to myapp-f6774f497-82w4r
`

Scale
`
kubectl -n dev-to scale deployment/myapp --replicas=2
`

Testando replicas
`
while true
do curl "http://dev.local/app/hello"
echo
sleep 2
done
`
Testando replicas com espera

`
while true
do curl "http://dev.local/app/wait"
echo
done
`

## Verificando a URL da aplicação
`minikube -p dev.to service -n dev-to myapp --url`

Verificando ip e a porta

`
curl -X GET http://dev.local/app/users
`

Adicionando novo usuário
`
curl --location --request POST 'http://dev.local/app/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "new user",
    "birthDate": "2010-10-01"
}'
`

## Debud da aplicação:

add   JAVA_OPTS: "-agentlib:jdwp=transport=dt_socket,address=*:5005,server=y,suspend=n"
 
Alterando CMD para ENTRYPOINT no Dockerfile

`
kubectl get pods -n=dev-to
`

`
kubectl port-forward -n=dev-to <pod_name> 5005:5005
`

## KubeNs e Stern

`
kubens dev-to
`

`
stern myapp
` 

## Inciando tudo

`make k:all`


## Referencias

https://kubernetes.io/docs/home/

https://minikube.sigs.k8s.io/docs/

## Comando

```
##List profiles
minikube profile list

kubectl top node

kubectl top pod <nome_do_pod>
```

# TP IOT 
Réalisé par : Ons Tliba - GL5 INSAT 
## Introduction: RabbitMQ
Durant ce TP, nous allons se baser sur le paradigme Publish/Subscrib afin de mettre en oeuvre une communication asynchrone dans un système d'objets connectés. 


Nous disposons alors de deux clients qui communiquent entre eux via un [brocker](https://pip.pypa.io/en/stable/). 


Pour se faire, nous utiliserons [RabbitMq](https://pip.pypa.io/en/stable/) et le langage de programmation [Golang](https://pip.pypa.io/en/stable/).


## Réalisation
```bash
#Création d'un nouveau network 
docker network create rabbit
#Execution du Container rabbit-1
docker run -d --rm --net rabbits --hostname rabbit-1 --name rabbit-1 rabbitmq:3.8 
#Effectuer le log afin de verifier du bon fonctionnement de RabbitMq
docker logs rabbit-1
```
Une fois tout va bien, nous allons se localisation à l'intérieur du container afin d'activer le plugin management pour pouvoir se connecter au Dashboard. 

```bash
#Se localiser dans le container 
docker exec -it rabbit-1 bash
#Lister les plugins 
rabbitmq-plugins list 
#Sortir du container et activer le port  15672
docker run -d --rm --net rabbits -p 8083:15672 --hostname rabbit-1 --name rabbit-1 rabbitmq:3.8
#Se localiser dans le container une nouvelle fois et activer rabbitmq_management
rabbitmq-plugins enable rabbitmq_management
```
Desormais, vous pouvez naviguer sur le Dashboard et se connecter avec guest/guest.

Passons mainetenant au code ! Dans un répertoire rabbitmq/applications/ nous allons créer deux répertoires consumer et publisher qui vont basculer le travail ultérieurement. 

Via une méthode Http Post, le client envoie périodiquement un message que le publisher va publier dans une queue et ensuite le consumer va consommer. (Voir le code)
Une fois le message est consommé, les roles vont se basculer. Le consumer enverra un message ver le producer pour le notifier. 

## Exécution 
```bash
#Executer le producer
docker build . -t aimvector/rabbitmq-publisher:v1.0.0
docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest -p 83:83 aimvector/rabbitmq-publisher:v1.0.0
#Executer le consumer
docker build . -t aimvector/rabbitmq-consumer:v1.0.0
docker run -it --rm --net rabbits -e RABBIT_HOST=rabbit-1 -e RABBIT_PORT=5672 -e RABBIT_USERNAME=guest -e RABBIT_PASSWORD=guest aimvector/rabbitmq-consumer:v1.0.0
```
# Intro

Bonjour à tous .. Ici, c'est le tp corrigé de bog. Ci-dessous, je vous explique dans les moindres détails comment j'ai dockeriser le tp pour qu'enfin tout puisse fonctionner et communiquer parfaitement ensemble.

## cloner le projet

Première des choses à faire : ouvrez une terminale et exécuter la commande suivante:

    git clone https://github.com/atemengue/tp-blog.git

Cette commande clonera le projet (devoir) dans un dossier appelé tp-blog.

> Rassurez-vous d'avoir git installé sur votre machine. Dans le cas contraire, téléchargez le code du devoir [**ici**](https://github.com/atemengue/tp-blog)

## explication de la structure du projet

Après avoir effectué les étapes précédentes, nous ouvrons le dossier tp-blog et nous aurons cette structure à l'intérieur :
![structure du projet](/assets/1.png)

Nous remarquons ici que nous avons 6 dossiers dont :

- **client** : qui est l'application react
- **comments**: qui est une application nodejs du services des commentaires qui sera exposé au port **4001**
- **event_bus**: qui est une application nodejs du services des event_bus qui sera exposé au port **4005**
- **moderation**: qui est une application nodejs du services des modération qui sera exposé au port **4003**
- **posts**: qui est une application nodejs du services des posts qui sera exposé au port **4000**
- **query**: qui est une application nodejs du services des qyery qui sera exposé au port **4002**

## configuration du dockerfile de chaque service

Pour créer les images de chaque services, rendez vouss dans chaque dossier des services et créez un fichier sans extension appelé **Dockerfile**

expemple:

![structure du projet](/assets/2.png)

voici le le code de configuration de notre image dans le **Dockerfile**:

![configuration du dockerfile](/assets/3.png)

    FROM node   //j'importe nodejs qui sera utilisé pour lancer le service
    WORKDIR /app    // je creer un espace de travail
    COPY . /app/        //je copie tout les fichier du service dans mon espace de travail
    RUN npm i       // je demande à docker d'exécuter npm i ou npm install pour installer toutes les dépendances de mon service
    EXPOSE 80   // je demande d'exposer mon service au port 80 de mon contenaire
    CMD [ "npm", "start" ]  // je demande ensuite d'exécuter npm start pour lancer mon application nodejs

## petite réfection

Comme vous le savez tout l'objectif ici, c'est de mettre toutes nos application nodejs en service et les faire communiquer ensemble chacun étant dans son contenaire

Pour faire cela, nous allons créer 5 images.

- **commentsimage**
- **event_busimage**
- **moderationimage**
- **postsimage**
- **queryimage**

En suite, nous allons créer 5 services.

- **commentsService**
- **event_busService**
- **moderationService**
- **postsService**
- **queryService**

> chaque fois que nous conteneurisons une application , docker le met dans une machine virtuelle différente de notre machine physique. Alors les mots tels que **localhost** ne pourront plus être utilisés pour faire référence au host locale de notre machine physique ..
> docker ne comprendra plus que les noms des services pour indexer les localhost des machines virtuelles.

Alors nous devons modifier chaque fichier **index.js** de chaque dossier en modifiant l'expression localhost auquel il fait référence.

Exemple pour **event_bus**:
![modification de localhost pour les noms des services](/assets/4.png)

> ici, j'ai remplacé les localhost par le nom des services auxquels ils faisaient référence
> Nous avons fait la même chose pour chaque application nodejs

## récupérer l'image de nodejs depuis le docker Hub

Pour faire cela, il suffit d'exécuter la commande suivante :

    sudo docker pull node

## créer une nouvelle image

Pour créer l'image, rendez-vous à l'emplacement de l'application correspondante à ce service et exécutez la commande suivante :

    sudo docker build -t appimage .

    //ici appimage est le nom de l'image et le "." Signifie que le Dockerfile est dans le même répertoire où je me trouve

# docker compose

## configuration du docker-compose

Rendez-vous /tp-blog et exécutez la commande suivante pour créer le dossier de docker-compose

    mkdir docker-compose.yml

Dans ce dossier, nous allons configurer le docker-compoose
Exemple:
![configuration du docker-compose](/assets/5.png)

    version: "3.5"  // représente la version du docker compose à utiliser
    services:       // ici je vais creer tous mes services
    postsService:   // je nom le service
        image: postsimage   //je le lie à une image
        ports:  // je le lie à un port
        - "4000:4000"   //ici je spécifie juste de lier le port 4000 de l'image au port 4000 de machine
        networks:   // je spécifie le réseau au quel il appartient
        - blog      // le nom du réseau en question

    commentsService:
        image: commentsimage
        ports:
        - "4001:4001"
        networks:
        - blog

    queryService:
        image: queryimage
        depends_on:     // étant donné que event_bus doit dabord s'exécuter avant query.. je fais dépendre query d'event_bus
        - event-busService
        ports:
        - "4002:4002"
        networks:
        - blog

    moderationService:
        image: moderationimage
        ports:
        - "4003:4003"
        networks:
        - blog

    event-busService:
        image: event-busimage
        ports:
        - "4005:4005"
        networks:
        - blog

    networks:   // ici je creer le réseau
    blog:

## lancer toutes mes applications

Pour faire cela, je me mets à la racine de mon projet "/tp-blog/" et j'exécute la commande suivante :

    docker-compose up

Et voilà tous fonctionne :)

# exécuter l'application react (client)

Rendez-vous dans le répertoire /tp-blog/client et exécutez les commande suivante

    npm i

Ceci installera les dépendances de react

    npm start

Ceci lancera l'application client et vous pouvez maintenant poster et commenter vos blog

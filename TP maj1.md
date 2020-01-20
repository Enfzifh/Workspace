# TP de RT 0704 : Une plateforme de code à la demande

## PARTIE 1 : Mise en place d'éléments de l'infrastructure

### Gestionnaire de file

Le gestionnaire de file de message RabbitMQ s'eécutera sur un système Linux. Vous prendrez la distribution de votre choix.

1. installation d'une machine virtuelle linux dans VirtualBox
   - la machine aura une interface réseau en NAT, et soit une seconde interface en réseau privé hôte, soit un forward du port 2222 de l'hôte vers le port 22 de l'invité
2. installez docker CE dans la machine virtuelle précédemment créée
3. téléchargez et installer l'image officielle docker de RabbitMQ
4. exécuter l'image rabbitMQ téléchargée
    - testez l'accès à l'interface WEB d'administration de RabbitMQ depuis la machine virtuelle
    - testez l'accès à l'interface WEB d'administration de RabbitMQ depuis la machine physique

### Mise en place de l'environnement de développement

1. Installez VSCode sur la machine physique
2. Installez le module *remote development* de VSCode
3. Mettez en place un échange de clé RSA entre la machine physique et la machine virtuelle
4. Connectez vous depuis VSCode à la machine virtuelle
5. Vérifiez que python3, pip3 et virtualenv / venv sont bien installés sur la machine virtuelle

### Producteur / consommateur dans RabbitMQ

Dans un premier temps, on va faire un programme python de lecture, écriture dans une file, selon un modèle producteur /consommateur.

1. créez un environnement virtuel sur la machine virtuelle
2. installez dans cet environnement virtuel les modules Flask et Pika
3. Connectez vous depuis VSCode sur l'hôte à l'invité, et accédez à l'environnement virtuel précédemment créé
4. Ecrivez une fonction de création d'une file nommée (*cf* cours)
5. Ecrivez une fonction d'écriture simple dans une file (*cf* cours), vous n'utiliserez pas de *fanout* juste une écriture directe
6.  Ecrivez une fonction de lecture simple dans une file (*cf* cours)
7.  testez les deux fonctions précédentes

### API REST d'accès à RabbitMQ

On va maintenant accéder aux fonctions précédemment développées, au travers de services REST. L'ensemble des données, paramètres des requêtes ou réponses seront en JSON

1. créez un serveur Flask sur l'invité
2. définissez les services suivants :
    - création d'une file :
      - route : */rabbit/*
      - méthode : POST
      - paramètre : nom de la file
    - dépôt d'un message dans une file
      - route : */rabbit/\<nom_file>/*
      - méthode : POST
      - paramètre : message
    - lecture d'un message dans une file
      - route : */rabbit/\<nom_file>/*
      - méthode : GET
3. écrivez des fonctions d'accès aux services, qui exploitent le module *requests*

### API GIT en python

Il est possible d'accéder, en python, aux dépôts GIT de deux manières :
1. en exécutant une commande système depuis python
2. en exploitant une API GIT

Dans le cadre de ce TP, on utilisera la seconde solution.

Avant de tester l'API GIT de python, créez sur la plate-forme GIT de votre choix un compte. Créez sur la plate-forme un projet.

1. Installez, dans votre environnement virtuel, le module python GitPython
2. Ecrivez une fonction permettant de cloner localement le contenu d'un dépôt

Vous pourrez exécuter la mise en place des fichiers sur le dépôt GIT de façon manuelle, si vous le souhaitez, vous pouvez écrire aussi les deux fonctions suivantes :

1. Ecrivez une fonction permetant d'ajouter un fichier au projet précédemment créé
2. Ecrivez une fonction permetant de mettre à jour un fichier du projet précédemment créé

### API Docker en Python

Comme dans le cas de l'API GIT, il est possible de piloter Docker depuis une API python.

1. Installez, dans votre environnement virtuel, le module python Docker
2. Ecrivez une fonction permetant d'exécuter un conteneur simple
3. Ecrivez une fonction permettant de construire un conteneur à partie d'un *dockerfile*
4. Ecrivez une fonction permettant de réupérer la liste des conteneurs en cours exécution

# Partie 2 : Infrastructure et calcul

Notre infrastructure sera composée :
- d'un conteneur Docker RabbitMQ contenant deux files par projet
  - la file ToDo
  - la file Done
- d'une instance Flask assurant l'accès au conteneur RabbitMQ (écriture et lecture dans les files)
- d'un compte GitHub ou GitLab
- d'un code commanditaire
- d'un code exécutant

## Opérations du commanditaire

Au préalable, vous aurez défini le *dockerfile* permettant de résoudre les différentes tâches du job à effectuer.

Les opérations assurés par le commanditaire sont :

1. mise dans le dépôt GIT du *dockerfile* qui correspond aux jobs à effectuer
2. génération des tâches à résoudre
   - chaque tâche à résoudre est représenté sous la forme d'une chaîne json comprenant au minimum
     - l'identifiant du projet
     - l'identifiant de la tâche
     - l'adresse GIT du dépôt contenant le *dockerfile* de résolution d'un job
     - la ligne de commande à exécuter pour la résolution d'une tâche (cette ligne comprends le script ainsi que les paramètres)
   - chacune des tâches (chaîne json) est stockée dans la file ToDo
3. la souscription à la file Done associée au projet
  - la récuperation des résultats
  - le calcul du résultat global

**Remarque** lors de la reconstruction du résultat, le commanditaire pourra contrôler les tâches déjà efectuées, et finaliser la reconstruction uen fois tous les résultats des jos récupérés.

## Opération d'un *worker*

Les opérations assurés par le *worker* sont :

1. récupération d'une tâche dans la file ToDo
2. mise en place de l'infrastructure Docker
  - récupération (clonage) du dépôt GIT
  - exécution du docker build
3. exécution de la commande dans l'environnement docker
4. émission du résultat vers la file Done
5. s'il reste dans tâches dans la file ToDo, retourner en 1



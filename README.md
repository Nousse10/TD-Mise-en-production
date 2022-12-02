# TD de mise en production : Déploiement avec Mogenius

Le but de ce TD est de réaliser des déploiements avec Mogenius. Nous commencerons par le déploiement d'une application web simple, puis nous déployerons notre application Java avec une base de données PostgreSQL.

## I - Déploiement d'une première application simple

L'objectif de cette première partie est de déployer une application simple sur un serveur distant. Pour cela, nous allons utiliser la plateforme Mogenius.
Mogenius est un outil de déploiement qui permet de déployer des applications sur des serveurs distants. Il est basé sur le principe de recette, c'est à dire que l'on va déployer une application sur un serveur de recette, puis sur un serveur de production. Il est possible de déployer plusieurs applications sur un même serveur.

### 1 - Création de l'application

Nous commençons par créer un simple fichier HTML. Il n'affichera qu'un titre sur la page. Nous l'enregistrons dans un dossier nommé *app* et nous l'appelons *index.html*.

### 2 - Build local de l'application

Nous allons maintenant créer un fichier Dockerfile qui va nous permettre de créer une image Docker de notre application. Pour cela, nous utilisons l'image Docker *nginx* qui est une image de serveur web. Nous copions notre dossier *app* contenant notre dichier *index.html* dans le dossier */usr/share/nginx/html* de l'image :

```Dockerfile
FROM nginx:alpine 
COPY app/ /usr/share/nginx/html
```

Nous pouvons maintenant build notre image Docker avec la commande suivante :
```bash
docker build -t html-server-image:v1 .
```

Une fois l'image créée, nous pouvons lancer un conteneur HTML à partir de cette image avec la commande suivante :
```bash
docker run -d -p 80:80 html-server-image:v1
```

Nous pouvons maintenant accéder à notre application en allant sur l'adresse suivante : http://localhost:80.

Nous obtenons bien notre page HTML, avec le titre *Bienvenue sur la page du TD de mise en production!* :

![Résultat du build local](/images/build_local_resultat.png "Page HTML")
*Figure 1 : Résultat du build local*

### 3 - Déploiement de l'application sur Mogenius

Nous allons maintenant déployer notre application sur Mogenius. Pour cela, nous avons créé un compte sur le site de Mogenius : https://mogenius.com. Il est également nécessaire de lier un compte Github à Mogenius afin de pouvoir déployer des applications depuis nos *repositories*.

Pour créer un projet sur Mogenius, nous cliquons sur *Create new Cloudspace*.
Il suffit ensuite de renseigner le nom interne du projet. Nous l'appelons *td-mise-en-prod*.

Nous allons ensuite ajouter un service à partir de notre propre code (option *Bring your own code* de la création). Nous renseignons le nom du service et lions ce dernier à notre *repository* Github, en renseignant la bonne branche. Nous ne changeons pas les paramètres de ressources par défaut car ce n'est pas important pour ce projet. Nous reviendrons sur ce sujet dans la partie suivante. Il est également nécessaire de renseigner le port d'écoute de l'application. Nous renseignons le port *80* car c'est le port par défaut de notre application.

Une fois créé, le service clone *repository* Github, build puis démarre automatiquement. Nous pouvons voir les logs du service associés depuis l'onglet *Logs* :

![Logs du service](/images/logs_service.png "Logs du service")
*Figure 2 : Onglet des logs du service, ici ceux du git clone*

Dès que le service est prêt, il est marqué comme *running* et nous pouvons accéder à notre application en cliquant sur *Hostname > External hostname*. Nous sommes renvoyés vers un nouvel onglet où la page HTML de notre application est bien affichée.

Dans cette première partie, nous avons déployé une application simple sur un serveur distant. Nous avons commencé par utiliser Docker pour créer une image de notre application. Puis, nous avons utilisé la plateforme Mogenius pour déployer notre application. 


## II - Déploiement de notre application Java avec PostgreSQL

Dans cette partie, nous allons déployer notre application Java du cours *Programmation full-stack* sur Mogenius. Nous allons également utiliser une base de données PostgreSQL pour stocker les données de notre application.

### 1 - Déploiement de la base de données PostgreSQL

Nous allons commencer par déployer notre base de données PostgreSQL sur Mogenius. Pour cela, nous allons créer un nouveau service. Nous allons partir du template gratuit *PostgreSQL*.

Nous renseignons le nom du service. Le port d'écoute de la base de données est 5432. Il s'agit du port par défaut de PostgreSQL. Nous le laissons tel quel.

Il est également nécessaire de porter attention aux paramètres de ressources. En effet, comme nous utlisons la version gratuite de Mogenius, nous avons un quota de ressources limité. La ressource limitante ici est le CPU, où 0.5 Core sont alloués au maximum. PostgreSQL ayant besoin d'un minimum de 0.3 Core pour fonctionner, il ne resterait que 0.2 Core pour notre application Java. Cela devrait fonctionner.

L'étape suivante est de configurer les variables d'environnements du service. PostgreSQL a besoin de plusieurs variables d'environnements pour fonctionner : *POSTGRES_PASSWORD*, *POSTGRES_USER* *POSTGRES_DB*. 

Pour la variable *POSTGRES_PASSWORD*, il est nécessaire de créer une variable secrète. Cela nous permet de ne pas afficher en clair le mot de passe de la base de données dans la configuration du service. Pour cela, nous allons dans l'onglet *Key Vault*. En cliquant sur *Add Secret*, il est demandé de renseigner un nom et une clé. Nous renseignons *database-password* comme nom et une valeur quelconque comme clé. 

![Variable secrète](/images/key_vault.png "Variable secrète")
*Figure 3 : Ajout d'une variable secrète*

Nous pouvons ensuite utiliser cette variable dans notre service en renseignant *database-password* comme valeur de la variable d'environnement *POSTGRES_PASSWORD* (en choisissant le type *Key Vault*). Pour les variables *POSTGRES_USER* et *POSTGRES_DB*, nous pouvons renseigner respectivement les valeurs *postgres* et *covid-db* (choisis arbitrairement).

Nous obtenons finalement la configuration suivante :
![Variables d'environnement](/images/variables_environnement_postgresql.png "Variables d'environnement")
*Figure 4 : Configuration des variables d'environnement du service PostgreSQL*

Une fois tous ces paramètres renseignés, nous pouvons valider la création du service. Ce dernier ce lance automatiquement et nous pouvons voir également les logs du service depuis l'onglet *Logs* afin de vérifier qu'il a bien démarré.

Il est à noter que lorsque nous changeons une variable d'environnement, un simple redémarrage du service de suffit pas. Il faut supprimer le service puis le recréer pour que les changements soient pris en compte.

### 2 - Déploiement de l'application Java

Nous allons maintenant déployer notre application Java. Pour cela, nous allons créer un dernier service. Nous partons évidemment cette fois-ci de notre propre code. 

La configuration est similaire à celle du premier service que nous avons créé pour notre application simple. Nous renseignons le nom du service et lions ce dernier à notre *repository* Github, en renseignant la bonne branche (master). 

Nous pouvons voir que le total de ressources allouables disponibles n'est plus que de 0,2 Core, car le srvice PostgreSQL prends déjà 0,3 Core. Ce sera suffisant pour déployer notre application Java.

Le port d'écoute de notre application est 8080. Nous renseignons ce port dans le champ *Port*.

Il est important de renseigner toutes les variables d'environnements suivantes, sans quoi notre application ne pourra pas se connecter à la base de données PostgreSQL :
- *DATABASE.HOST* : Nom d'hôte interne du service PostgreSQL. Il est à récupérer en cliquant sur *Hostname* dans la page du service. La valeur est ici *postgre-db-c40tya*
- *DATABASE.USERNAME* : Nom d'utilisateur de la base de données PostgreSQL. Il a été renseigné dans les variables d'environnement du service PostgreSQL. La valeur est *postgres*
- *DATABASE.PORT* : Port d'écoute de la base de données PostgreSQL. Il a aussi été renseigné dans les variables d'environnement du service PostgreSQL (valeur par défaut). La valeur est *5432*
- *DATABASE.PASSWORD* : Mot de passe de la base de données PostgreSQL. En choisissant le type *Key Vault*, la valeur est *database-password* (le nom de la variable secrète que nous avons créé précédemment)
- *DATABASE.NAME* : Nom de la base de données PostgreSQL. Renseigné également dans l'autre service, la valeur est *covid-db*

Nous obtenons finalement la configuration suivante :
![Variables d'environnement](/images/variables_environnement_java.png "Variables d'environnement")
*Figure 5 : Configuration des variables d'environnement du service Java*

Nous pouvons valider la création du service. Ce dernier effectue les mêmes étapes que le premier service, c'est-à-dire qu'il effectue un git clone du repository Github, build l'image Docker puis démarre (docker run). Nous pouvons voir les logs du service depuis l'onglet *Logs* afin de vérifier qu'il s'est bien lancé.

En accédant à l'URL du service depuis *Hostname > External hostname*, nous pouvons voir que notre application Java est bien déployée et fonctionnelle. Par exemple, en accédant à l'URL */public/centers* qui correspond à la méthode HTTP GET de récupération de tous les centres, nous obtenons la réponse suivante :
![Déploiement de l'application Java](/images/deploiement_java.png "Déploiement de l'application Java")
*Figure 6 : Résultat de la requête /public/centers récupérant tous les centres*

Voici la requête en question : https://covid-api-prod-td-mise-en-prod-zst38k.mo6.mogenius.io/public/centers

Nous observons que nous ne récupérons finalement aucun centre. C'est normal, car notre base de données est vide.

En conclusion de cette partie, nous avons déployé une application Java et une base de données PostgreSQL sur Mogenius. Nous avons pu voir que l'application Java pouvait se connecter à la base de données PostgreSQL et récupérer des données.
____

Pour conclure ce TD, nous avons vu comment déployer une application web simple dans un premier temps, puis une application Java et une base de données PostgreSQL sur Mogenius. Lorsque le code source de l'application sera mis à jour, il suffira de relancer le build du service Java pour que les changements soient pris en compte.
Malheureusement, comme nous étions limités par le nombre de ressources disponibles de part la gratuité de l'offre, nous n'avons pas pu déployer le front-end de notre application web.
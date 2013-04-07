#TutoHeroku

Ce tutoriel vous présente la marche à suivre pour réussir à déployer les bots du projet PokeBattle sur Internet.
L'intérêt de pouvoir déployer sur le net est que vos pokémons, vos juges et vos infirmières Joëlle deviennent 
totalement autonomes de votre environnement de développement. Une fois le projet terminé, vous pourrez laisser vos 
bots continuer à vivre leur vie sans avoir à vous en occuper.

Si vous voulez plus d'informations, vous pouvez consulter les deux tutoriels qui m'ont servi de base : 
- [Getting started with Heroku](https://devcenter.heroku.com/articles/quickstart)
- [Getting Started with Java on Heroku](https://devcenter.heroku.com/articles/java)

Remarquez aussi que le présent tutoriel est fait pour fonctionner sur les machines de l'IUT où le client en ligne de 
commande d'héroku est déjà installé. Si vous voulez l'installer sur votre machine il vous suffit de le télécharger à 
l'adresse suivante : (https://toolbelt.heroku.com/). De plus si vous utilisez des OS exotiques comme Windows (je ne 
ferai pas de commentaires désobligeant), il vous faudra adapter les commandes données pour que cela fonctionne chez 
vous. 

##Heroku

Heroku est un service de cloud computing de type PAaS (Platform As a Service). Dans ce type d'environnement le 
fournisseur de service met à disposition de ses clients une plate-forme d'exécution d'application. Cette plate-forme met 
à disposition les ressources physiques (serveurs), l'infrastructure (réseau, stockage et sauvegarde) ainsi que l'ensemble
des logiciels nécessaires au lancement des applications clientes (systèmes d'exploitation, langage de programmation, outils 
de développements, ...). 

Le client bénéficie d'un environnement de développement managé, hébergé, et maintenu par un prestataire extérieur à son 
entreprise. Le principal attrait de ses environnements vis à vis des hébergements plus classiques, est qu'ils proposent 
une solution souvent simple d'accès capable de s'adapter facilement à la fluctuation des besoins des applications. Par 
exemple si une application subit une montée en charge temporaire, l'utilisateur pourra augmenter les ressources mises à 
dispositions de son application. La facturation se fait en fonction de l'usage. Ainsi plus une application demandera de 
temps de calcul, de stockage ou de mémoire, plus on payera chèr. À l'inverse si notre application n'est plus utilisée, le
coût de l'hébergement sera nul. L'idée est de ne payer que pour ceux dont on a besoin et non de louer un serveur que 
l'on utilisera globalement pas ou peu mais que l'on aura dimensionné pour être capable de supporter une charge maximale 
théorique.

Coté bases de données, Heroku a intégré plusieurs bases NoSQL comme MongoDB ou Redis. Par défaut vous avez accès à une 
base relationnelle PostgreSQL. Vos applications utilisant JPA, vous pourrez l'utiliser simplement en configurant votre 
fichier `persistence.xml`.

En plus de proposer un hébergement gratuit pour nos bots, Heroku dispose d'un avantage supplémentaire qui est que le 
déploiement se fait grâce à `git`. La mise en ligne d'une nouvelle version de votre application se fera 
grâce à un simple `git push`.

##Fonctionnement

L'architecture Heroku est centrée sur les processus au sens Unix du terme. Le code déployé est analysé pour détecter le 
type d'application, ainsi que les processus nécessaires ; l'analyse déclenche la mise en place d'un environnement dédié.
Par exemple quand on pousse une application Java, une JVM est d'abord déployée puis c'est Maven qui s'occupe de construire
l'application avant de lancer les processus configurés par l'utilisateur.

Les processus sont contenus dans des conteneurs léger appelés *Dyno*. Un *Dyno* lance une unique commande passée par 
l'utilisateur. Les commandes exécutées dans les *Dynos* peuvent être de deux types : les processus *Web*, les processus 
*Workers* (tache de fond). Les types de processus et les commandes utilisées sont déclarées dans le fichier `Procfile` 
qui sera placé à la racine du projet.

## Configuration *git*
Heroku s'appuyant sur `git`, la première chose à faire est donc d'avoir un dépôt convenablement configuré.

La seconde étape est de se créer un compte sur le site [heroku.com](https://id.heroku.com/signup/get).

La connexion au dépôt distant se fait en utilisant `ssh`, pour l'utiliser il faut configurer convenablement vos clefs. 
Sur les machines de l'IUT, les clefs ne peuvent pas être sauvegardées sur le dossier `net-home`, il faudra donc refaire 
cette manip avant chaque TP.

```sh
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
$ heroku keys:add
Uploading ssh public key /home/nedjar/.ssh/id_rsa.pub
```

Une fois cette étape effectuée, vous pouvez vous connecter dans un terminal en tapant la commande suivante :
```sh
$ heroku login
Enter your Heroku credentials.
Email: plopplop+heroku@gmail.com
Password (typing will be hidden): 
``` 
## Configuration *Maven*
Quand l'on déploie une application Java, le runtime d'Heroku utilise Maven pour construire votre projet et récupérer 
toute ses dépendances. Pour que ces dépendances soient placées dans un dossier accessible, il faut rajouter à votre 
fichier `pom.xml` les lignes suivantes :
```XML
 <build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>2.4</version>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals><goal>copy-dependencies</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Attention, Eclipse vous indiquera une erreur dans votre `pom.xml` quand vous aurez rajouter ces lignes. 
Cliquez simplement sur l'erreur et demandez simplement à Eclipse de ne plus vous la signaler.
Ce problème vient du plugin m2e qui ne sait pas gérer la copie des dépendances. Comme nous allons tester notre application en ligne de commande, cette erreur ne sera pas gênante pour nous.

## Test de l'application localement
Pour vérifier que notre application est correctement configurée, nous allons lancer les commandes suivantes :
```sh
$ mvn package
$ java -cp target/classes:"target/dependency/*" fr.univaix.iut.pokebattle.run.PokemonMain
```
Logiquement si tout va bien vous devriez avoir une sortie semblable à cela :
```sh
624 [main] INFO com.twitter.hbc.httpclient.BasicClient - New connection executed: hosebird-client-0, endpoint: /1.1/user.json?delimited=length&stall_warnings=true
838 [hosebird-client-io-thread-0] INFO com.twitter.hbc.httpclient.ClientBase - hosebird-client-0 Establishing a connection
2066 [hosebird-client-io-thread-0] INFO com.twitter.hbc.httpclient.ClientBase - hosebird-client-0 Processing connection data
2078 [pool-3-thread-1] INFO fr.univaix.iut.pokebattle.tuse.UserStreamAdapter - Unimplemented event handler: onFriendList
...
```
## Déclaration des processus
Pour indiquer les processus à créer ainsi que les commandes à utiliser, vous devez créer un fichier nommé `Procfile` à 
la racine de votre projet. Le contenu de ce fichier sera le suivant :
```sh
worker: java -cp target/classes:target/dependency/* fr.univaix.iut.pokebattle.run.PokemonMain
```

Évidement si la classe contenant votre méthode `main` n'est pas `fr.univaix.iut.pokebattle.run.PokemonMain`, vous devrez 
adapter cet exemple pour correspondre à votre cas. Remarquez que contrairement au test local ou nous avions dû protéger notre chaine par des quotes, dans ce fichier il ne faut pas le faire.

## Déploiement de l'application

Avant de pouvoir déployer votre application, il faut la créer avec la commande `heroku` :
```sh
heroku create pokemon
Enter your Heroku credentials.
Email: plopplop+heroku@gmail.com
Password (typing will be hidden): 
Creating pokemon... done, stack is cedar
http://pokemon.herokuapp.com/ | git@heroku.com:pokemon.git
Git remote heroku added
``` 
Dans l'exemple ci-dessus, `pokemon` représente le nom de votre application sur Heroku. Ce paramètre est optionnel mais 
je ne saurai trop vous conseiller de toujours l'utiliser. 

Le déploiement se fait ensuite simplement comme ceci :
```sh
$ git push heroku master
Counting objects: 47, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (25/25), done.
Writing objects: 100% (47/47), 10.25 KiB, done.
Total 47 (delta 19), reused 42 (delta 17)

-----> Heroku receiving push
-----> Java app detected
-----> Installing OpenJDK 1.6... done
-----> Installing Maven 3.0.3... done
-----> Installing settings.xml... done
-----> executing /app/tmp/repo.git/.cache/.maven/bin/mvn -B -Duser.home=/tmp/build_3k0p14ghrmdzs -Dmaven.repo.local=/app/tmp/repo.git/.cache/.m2/repository -s /app/tmp/repo.git/.cache/.m2/settings.xml -DskipTests=true clean install
       [INFO] Scanning for projects...
       [INFO]                                                                         
       [INFO] ------------------------------------------------------------------------
       [INFO] Building pokebot 1.0-SNAPSHOT
       [INFO] ------------------------------------------------------------------------
       ...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 10.062s
       [INFO] Finished at: Tue Jan 31 23:27:20 UTC 2012
       [INFO] Final Memory: 12M/490M
       [INFO] ------------------------------------------------------------------------
-----> Discovering process types
       Procfile declares types -> worker
-----> Compiled slug size is 948K
-----> Launching... done, v3
       http://pokemon.herokuapp.com/ deployed to Heroku
```

Quand vous avez exécuté la commande `heroku create pokemon`, elle rajoute un *remote* appelé `heroku` dans lequel vous 
pouvez pusher. La configuration de ce remote est située dans le fichier `.git/config`. Elle devrait ressembler à ceci :
```
[remote "heroku"]
	url = git@heroku.com:pokemon.git
	fetch = +refs/heads/*:refs/remotes/heroku/*
```

Par défaut, Heroku est configuré pour lancer uniquement un seul processus de type web. Pour lancer notre worker, exécutez la commande suivante :
```sh
 $ heroku ps:scale worker=1
Scaling worker processes... done, now running 1
``` 
Pour vérifier que votre programme est bien lancé, utilisez la commande suivante :
```sh
$ heroku ps
=== worker: `java -cp target/classes:"target/dependency/*" fr.univaix.iut.pokebattle.run.PokeTimerMain `
worker.1: up 2013/04/03 03:51:22 (~ 23m ago)
``` 

Si vous voulez plus d'informations sur votre application, vous pouvez afficher les logs avec la commande suivante :

```sh
$heroku logs
2013-04-01T21:33:51+00:00 heroku[api]: Enable Logplex by plopplop+heroku@gmail.com
2013-04-01T21:36:04+00:00 heroku[api]: Release v1 created by plopplop+heroku@gmail.com
2013-04-01T21:36:05+00:00 heroku[api]: Deploy 3d9f3bf by plopplop+heroku@gmail.com
2013-04-01T21:36:05+00:00 heroku[slugc]: Slug compilation finished
2013-04-01T21:37:36+00:00 heroku[api]: Scale to worker=1 by plopplop+heroku@gmail.com
2013-04-01T21:37:41+00:00 heroku[worker.1]: Starting process with command `java -cp target/classes:"target/dependency/*" fr.univaix.iut.pokebattle.run.PokemonMain `
2013-04-01T21:37:42+00:00 heroku[worker.1]: State changed from starting to up
2013-04-01T21:37:42+00:00 app[worker.1]: 570 [main] INFO com.twitter.hbc.httpclient.BasicClient - New connection executed: hosebird-client-0, endpoint: /1.1/user.json?delimited=length&stall_warnings=true
2013-04-01T21:37:42+00:00 app[worker.1]: 660 [hosebird-client-io-thread-0] INFO com.twitter.hbc.httpclient.ClientBase - hosebird-client-0 Establishing a connection
2013-04-01T21:37:42+00:00 app[worker.1]: 1129 [hosebird-client-io-thread-0] INFO com.twitter.hbc.httpclient.ClientBase - hosebird-client-0 Processing connection data
2013-04-01T21:37:42+00:00 app[worker.1]: 1138 [pool-3-thread-1] INFO fr.univaix.iut.pokebattle.tuse.UserStreamAdapter - Unimplemented event handler: onFriendList
```
## Paramétrage de la base de données
Toute application sur Heroku a accès gratuitement à une base de données *PostgresSQL* dès sa création. Cette base de 
données a une capacité bien évidement très limitée (10000 tuples et 5.9MB). Pour notre besoin, cela suffira amplement.
Pour avoir les informations sur cette base de données vous pouvez taper la commande suivante :
```sh
$ heroku  pg:info
=== HEROKU_POSTGRESQL_GOLD_URL (DATABASE_URL)
Plan:        Dev
Status:      available
Connections: 1
PG Version:  9.1.9
Created:     2013-04-01 04:36 UTC
Data Size:   5.9 MB
Tables:      0
Rows:        0/10000 (In compliance)
Fork/Follow: Unsupported
```
Comme vous le voyez nous avons une seule base qui s'appelle `HEROKU_POSTGRESQL_GOLD_URL`. Pour récupérer l'URL de 
connexion à cette BD dans nos programmes, Heroku configure une variable d'environnement `DATABASE_URL`. La commande 
`heroku config` permet d'afficher toutes les variables d'environnement qui seront disponibles quand notre application 
sera déployée.

```sh
DATABASE_URL:               postgres://wqkhatrxxl:Tad6C_6Snb9ZqrDSyOv2RGC2@ec2-107-22-169-102.compute-1.amazonaws.com:5432/dc9cle3o9vvq
HEROKU_POSTGRESQL_GOLD_URL: postgres://wqkhatrxxl:Tad6C_6Snb9ZqrDSyOv2RGC2@ec2-107-22-169-102.compute-1.amazonaws.com:5432/dc9cle3o9vvq
JAVA_OPTS:                  -Xmx384m -Xss512k -XX:+UseCompressedOops
MAVEN_OPTS:                 -Xmx384m -Xss512k -XX:+UseCompressedOops
PATH:                       /app/.jdk/bin:/usr/local/bin:/usr/bin:/bin
```
Si vous voulez passer d'autres variables à vos programmes regardez la documentation de cette commande en faisant : `heroku help config`.

Malheureusement pour nous, l'URL de connexion n'est pas une URL jdbc standard. Il va donc falloir au début de notre 
programme récupérer la variable d'environnement `DATABASE_URL`, la découper et la passer à JPA pour qu'il puisse construire
correctement notre objet `EntityManager`. La configuration de notre connexion ira donc en partie dans notre fonction `main()` 
et plus uniquement dans le fichier `persistence.xml`. Bien évidement ça n'aura pas d'incidence sur la base de test qui 
restera une base embarquée totalement indépendante de l'environnement technique.

Le gros avantage de la configuration que nous allons présenter est que vous pourrez vous connecter à votre BD située sur
Heroku même quand vous essaierez votre application localement.

Si jamais pour une raison ou une autre par la suite vous avez besoin de réinitialiser votre BD vous pouvez le faire grâce 
à la commande `heroku pg:reset`.

Pour pouvoir faire fonctionner votre programme de la même manière en local que sur Heroku, il faut définir la variable 
`DATABASE_URL` dans Eclipse avec la valeur précédente. Pour ce faire allez dans le menu `Run->Run Configuration ...` 
choisissez votre main, aller dans l'onglet `environment`, cliquez sur `New` puis faite un copier/coller de la valeur 
ci-dessus.

### Récupération de la variable d'environnement :

La première tache que devra faire votre programme sera de récupérer la variable d'environnement `DATABASE_URL` avec la 
méthode `System.getenv()`:
```java
String databaseUrl = System.getenv("DATABASE_URL");
```

### Analyse de l'URL de connexion :
Ensuite il faut parser l'ancienne URL pour la transformer en une chaîne de connexion JDBC valide :

```java
URI dbUri = new URI(databaseUrl);
String username = dbUri.getUserInfo().split(":")[0];
String password = dbUri.getUserInfo().split(":")[1];
String dbUrl = "jdbc:postgresql://" + dbUri.getHost() + ':' + dbUri.getPort() + dbUri.getPath() + "?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory";
```

### Paramétrage de l'unité de persistance à l'exécution :
Pour passer des paramètres supplémentaires pour la construction d'une unité de persistance, la méthode statique 
`Persistence.createEntityManagerFactory()` a une surcharge comportant deux paramètre, le premier est le nom d'une unité 
de persistance définie dans le fichier `persistence.xml` et le second une `Map` permettant de passer les valeurs des 
paramètres que l'on a besoin de rajouter à l'exécution. Dans notre cas, nous avons besoin de définir les paramètres 
relatifs à la connexion JDBC : ``
```java
Map<String, String> props = new HashMap<String, String>();
props.put("javax.persistence.jdbc.driver", "org.postgresql.Driver");
props.put("eclipselink.target-database", "PostgreSQL");
props.put("javax.persistence.jdbc.url", dbUrl);
props.put("javax.persistence.jdbc.user", username);
props.put("javax.persistence.jdbc.password", password);
```

Une fois cette objet `Map` correctement construit la suite de la construction de notre `entityManager` se passe comme suit :
```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("pokebattlePU", props);
EntityManager em = emf.createEntityManager();
```
Le fichier `persistence.xml` se retrouve donc expurgé des paramètres devenus inutile : 
```XML
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">
    <persistence-unit name="pokebattlePU" transaction-type="RESOURCE_LOCAL">
        <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
        <properties>
            <property name="eclipselink.logging.level" value="INFO"/>
            <property name="eclipselink.ddl-generation" value="create-tables"/>
            <property name="eclipselink.ddl-generation.output-mode" value="database"/>
        </properties>
    </persistence-unit>
</persistence>
```

Pour finir, il faut aussi rajouter dans le fichier `pom.xml` la dépendance vers le connecteur JDBC de PostgreSQL :
```XML
<dependency>
    <groupId>postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>9.0-801.jdbc4</version>
</dependency>
```

###Amélioration de la solution présentée
Dans la solution présentée, la récupération de l'entityManager ne peut être faite qu'à partir de la fonction `main()` 
car pour que notre code reste testable unitairement, il ne faut pas que nos classes dépendent d'une ressource propre à 
l'environnement d'exécution. Il faudra donc que cet entityManager puisse être transmis à tous les DAO pour une utilisation
ultérieure. Dans la solution que nous allons présenter, nous allons faire une classe `DAOFactoryJPA` qui aura la 
responsabilité de créer tous les DAO à partir de l'objet `entityManager` qu'elle conservera. 
```java
public class DAOFactoryJPA {
		private static EntityManager entityManager;

		public static synchronized setEntityManager(EntityManager entityManager){
			DAOFactoryJPA.entityManager = entityManager;
		}

		public DAOPokemon createDAOPokemon(){
				return new DAOPokemonJPA(entityManager);
		}

		public DAOAttack createDAOAttack(){
				return new DAOAttackJPA(entityManager);
		}

		public DAOCombat createDAOCombat(){
				return new DAOCombatJPA(entityManager);
		}
}
```
La responsabilité de la méthode `main()` sera de construire l'entityManager et de le transmettre à la classe `DAOFactoryJPA` 
pour que tous les DAO soient créés avec les bons paramètres :

```java
private static Map<String, String> createConfigurationMap() throws URISyntaxException {
    URI dbUri = new URI(System.getenv("DATABASE_URL"));
    String username = dbUri.getUserInfo().split(":")[0];
    String password = dbUri.getUserInfo().split(":")[1];
    String dbUrl = "jdbc:postgresql://" + dbUri.getHost() + ':' + dbUri.getPort() + dbUri.getPath() + "?ssl=true&sslfactory=org.postgresql.ssl.NonValidatingFactory";

    Map<String, String> props = new HashMap<String, String>();
    props.put("javax.persistence.jdbc.driver", "org.postgresql.Driver");
    props.put("eclipselink.target-database", "PostgreSQL");
    props.put("javax.persistence.jdbc.url", dbUrl);
    props.put("javax.persistence.jdbc.user", username);
    props.put("javax.persistence.jdbc.password", password);
    return props;
}

public static void main(String[] args) throws URISyntaxException {
    Map<String, String> props = createConfigurationMap();

    EntityManagerFactory emf = Persistence.createEntityManagerFactory("pokebattlePU", props);
    EntityManager em = emf.createEntityManager();

    DAOFactoryJPA.setEntityManager(em);
   	// Suite du programme ...
}
```

Pour que nos tests soient eux aussi de la construction de notre unité de persistance, il faudra que chacune de vos 
classe de test contiennent une méthode d'initialisation appelée avant tous les tests :
```java
@BeforeClass
public static void initTestFixture() throws Exception {
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("pokebattlePUTest");
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    DAOFactoryJPA.setEntityManager(em);
    //... suite des initialisations pour les tests
}
```

Une fois cette amélioration faite, vous pourrez faire tourner correctement votre application en local, sur Heroku et lancer 
les tests en utilisant bien une base de données embarquée.


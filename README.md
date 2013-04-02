#TutoHeroku

Ce tutoriel vous présente la marche à suivre pour réussir à déployer les bots du projet PokeBattle sur Internet.
L'intérêt de pouvoir déployer sur le net est que vos pokémons, vos juges et vos infirmières Joëlle deviennent 
totalement autonomes de votre environnement de développement. Une fois le projet terminé, vous pourrez laisser vos 
bots continuer à vivre leur vie sans avoir à vous en occuper.

##Heroku

Heroku est un service de cloud computing de type PAAS (Platform As A Service). Dans ce type d'environnement le 
fournisseur de service met à disposition de ses clients une plate-forme d'exécution d'application. Cette plate-forme met 
à disposition les ressources physiques (serveurs), l'infrastructure (réseau, stockage et sauvegarde) ainsi que l'ensemble
des logiciels nécessaires au lancement des applications clientes (systèmes d'exploitation, langage de programmation, outils 
de développements, ...). 

Le client bénéficie d'un environnement de développement managé, hébergé, et maintenu par un prestataire extérieur à son 
entreprise. Le principal attrait de ses environnements vis à vis des hébergements plus classiques, est qu'ils proposent 
une solution souvent simple d'accès capable de s'adapter facilement à la fluctuation des besoins des applications. Par 
exemples si une application subit une montée en charge temporaire, l'utilisateur pourra augmenter les ressources mises à 
dispositions de son application. La facturation se fait en fonction de l'usage. Ainsi plus une application demandera de 
temps de calcul, de stockage ou de mémoire, plus on payera chère. À l'inverse si notre application n'est plus utilisée, le
coût de l'hébergement sera nul. L'idée est de ne payer que pour ceux que l'on a besoin et non de louer un serveur que 
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
adapter cet exemple pour correspondre à votre cas.

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

Le déploiement se fait ensuite simplement comme ceci :
```
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

Par defaut, Heroku est configuré pour lancer uniquement un seul processus de type web. Pour lancer notre worker, exécutez la commande suivante :
```
 $ heroku ps:scale worker=1
Scaling worker processes... done, now running 1
``` 
Pour vérifier que votre programme est bien lancé, utilisez la commande suivante :
```
$ heroku ps
=== worker: `java -cp target/classes:"target/dependency/*" fr.univaix.iut.pokebattle.run.PokeTimerMain `
worker.1: up 2013/04/03 03:51:22 (~ 23m ago)
``` 

Si vous voulez plus d'informations sur votre application, vous pouvez afficher les logs avec la commande suivante :

```
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



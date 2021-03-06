# README #

## Prérequis ##

Pour pouvoir démarrer le projet il est nécessaire d'installer les programmes
suivant:

* Java 8
* gradle
* Apache Tomcat v8.0.33
* mysql-5.6

Nous considérons que vous utilisez un OS basé sur Unix.

## Java

On commence donc par vérifier que java 8 est bien installée:

```bash
 java -version
 ```

Votre output devrait ressembler à cela:

```bash 
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)
```

Si l'ouptut n'affiche pas la première ligne avec "1.8" il faut installer la bonne version de java:

Suivez les instructions sur le site officiel d'Oracle:

[Oracle - java 8](https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)

## Gradle

Ensuite il faut faire de même pour gradle:

```bash
gradle -v
```

Votre output devrait ressembler à cela:


```bash
------------------------------------------------------------
Gradle 2.5
------------------------------------------------------------
```

Si ce n'est pas le cas il suffit de l'installer:

```bash
sudo apt-get install gradle
```

## Tomcat

Pour Apache Tomcat télécharger 
[tomcat](http://apache.cu.be/tomcat/tomcat-8/v8.0.33/bin/apache-tomcat-8.0.33.zip)
et dézippez-le.

## MySQL

Afin d'installer le projet, il faut que le serveur possède une base de données mysql.

Si ce n'est pas le cas, il suffit de l'installer :
```bash
sudo apt-get install mysql-server-<version>
```

Un script de création des utilisateurs ainsi que destables est disponibles ici au format MySQL: 
[creatikzDatabase.sql](https://gitlab.com/INFOF307_Groupe2/CreaTikZ/blob/master/resources/creatikzDatabase.sql)

Il crée notamment l'utilisateur test (pwd test) afin de pouvoir tester les fonctionnalités directement sans devoir passer par l'étape de création d'un comtpe.


## Projet

Ne pas oublier de cloner le projet:

```bash
git clone https://gitlab.com/INFOF307_Groupe2/CreaTikZ.git
```

# Prêt à lancer #

Pour les commandes suivantes, placez vous à la racine du projet.

## Démarrer l'application

```bash
bash creatikz app
```

## Démarrer le serveur web

```bash
bash creatikz server <chemin/vers/apachetomcat>
```

Une erreur peut survenir parfois par manque de permissions, changer ces permissions:

```bash
chmod +x <chemin/vers/apachetomcat>/bin/catalina.sh
```

## Pour arrêter le serveur web

```bash
bash creatikz shutserver <chemin/vers/apachetomcat>
```

###Aide
Si vous rencontrez des difficultés avec le lancement de l'application envoyez un mail à Groupe2@ulb.ac.be.

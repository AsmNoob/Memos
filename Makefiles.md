Introduction to Makefiles
=========================
[www.developpez.com](http://gl.developpez.com/tutoriel/outil/makefile/)

Exemple minimal
---------------

	all: hello

	hello: hello.o main.o
		gcc -o hello hello.o main.o

	hello.o: hello.c
		gcc -o hello.o -c hello.c -W -Wall -ansi -pedantic

	main.o: main.c hello.h
		gcc -o main.o -c main.c -W -Wall -ansi -pedantic

	clean:
		rm -rf *.o

	mrpropre: clean
		rm -rf hello

Commentaires:

- all: regroupe l'ensemble des executables à produire
- clean: supprime tous les fichiers intermédiaires
- mrpropre: supprime tout et permet une reconstruction complète du projet


Variables
---------

###Variables personnalisées###

	CC=gcc // compilateur
	CFLAGS=-W -Wall -ansi -pedantic // options de compilation
	LDFLAGS= // 
	EXEC=hello

	all: $(EXEC)

	hello: hello.o main.o
		$(CC) -o hello hello.o main.o $(LDFLAGS)

	hello.o: hello.c
		$(CC) -o hello.o -c hello.c $(CFLAGS)

	main.o: main.c hello.h
		$(CC) -o main.o -c main.c $(CFLAGS)

	clean:
		rm -rf *.o

	mrproper: clean
		rm -rf $(EXEC)


###Variables internes###

Différentes variables:

- $@: Nom de la cible
- $<: Nom de la première dépendance
- $^: Liste des dépendances
- $?: Liste des dépendances plus récentes que la cible
- $*: Nom du fichier sans suffixes

Exemple de code:

	all: $(EXEC)

	hello: hello.o main.o
		$(CC) -o $@ $^ $(LDFLAGS) // == $(CC) -o hello hello.o main.o $(LDFLAGS)

	hello.o: hello.c
		$(CC) -o $@ -c $< $(CFLAGS) // == $(CC) -o hello.o -c hello.c $(CFLAGS)

	main.o: main.c hello.h
		$(CC) -o $@ -c $< $(CFLAGS) // == $(CC) -o main.o -c main.c $(CFLAGS)

	clean:
		rm -rf *.o

	mrproper: clean
		rm -rf $(EXEC)


Règles d'inférences
-------------------

Règles génériques:
Exemple:

Construire un .O à partir d'un .c :::> %.o: %.c  => commandes
										

	EXEC=hello

	all: $(EXEC)

	hello: hello.o main.o
		$(CC) -o $@ $^ $(LDFLAGS)

	%.o: %.c
		$(CC) -o $@ -c $< $(CFLAGS)

	clean:
		rm -rf *.o

	mrproper: clean
		rm -rf $(EXEC)

Comme le montre clairement l'exemple précédent, main.o n'est plus reconstruit si hello.h est modifié. Il est possible de préciser les dépendances séparément des règles d'inférence et de rétablir le fonctionnement original, pour obtenir finalement : 

	EXEC=hello

	all: $(EXEC)

	hello: hello.o main.o
		$(CC) -o $@ $^ $(LDFLAGS)

	main.o: hello.h

	%.o: %.c
		$(CC) -o $@ -c $< $(CFLAGS)

	clean:
		rm -rf *.o

	mrproper: clean
		rm -rf $(EXEC)


Génération de la liste des fichiers objets
------------------------------------------

 Plutôt que d'énumérer la liste des fichiers objets dans les dépendances de la règle de construction de notre exécutable, il est possible de la générer automatiquement à partir de la liste des fichiers sources. Pour cela nous rajoutons deux variables au Makefile :

- SRC qui contient la liste des fichiers sources du projet.
- OBJ pour la liste des fichiers objets.

La variable OBJ est remplie à partir de SRC de la manière suivante :

	OBJ= $(SRC:.c=.o)

Exemple:

	CC=gcc
	CFLAGS=-W -Wall -ansi -pedantic
	LDFLAGS=
	EXEC=hello
	SRC= hello.c main.c
	OBJ= $(SRC:.c=.o)

	all: $(EXEC)

	hello: $(OBJ)
		$(CC) -o $@ $^ $(LDFLAGS)

	main.o: hello.h

	%.o: %.c
		$(CC) -o $@ -c $< $(CFLAGS)

	.PHONY: clean mrproper

	clean:
		rm -rf *.o

	mrproper: clean
		rm -rf $(EXEC)


Construction de la liste des fichiers sources
--------------------------------------------

	SRC= $(wildcard *.c) // obligation d'utiliser wildcard pour utiliser "*."

Commandes silencieuses
----------------------

Lorsque le nombre de règles d'un Makefile augmente, il devient très rapidement fastidieux de trouver les messages d'erreur affichés au milieu des lignes de commandes. Les Makefiles permettent de désactiver l'écho des lignes de commandes en rajoutant le caractère @ devant la ligne de commande

	all: $(EXEC)

	hello: $(OBJ)
		@$(CC) -o $@ $^ $(LDFLAGS)

	main.o: hello.h

	%.o: %.c
		@$(CC) -o $@ -c $< $(CFLAGS)

	.PHONY: clean mrproper

	clean:
		@rm -rf *.o

	mrproper: clean
		@rm -rf $(EXEC)


Les makefiles conditionnels
---------------------------

Supposons, par exemple, que nous souhaitions compiler notre projet tantôt en mode debug, tantôt en mode release sans avoir à modifier plusieurs lignes du Makefile pour passer d'un mode à l'autre. Il suffit de créer une variable DEBUG et tester sa valeur pour changer de mode : 

	DEBUG=yes
	CC=gcc
	ifeq ($(DEBUG),yes)
		CFLAGS=-W -Wall -ansi -pedantic -g
		LDFLAGS=
	else
		CFLAGS=-W -Wall -ansi -pedantic
		LDFLAGS=
	endif
	EXEC=hello
	SRC= $(wildcard *.c)
	OBJ= $(SRC:.c=.o)

	all: $(EXEC)
	ifeq ($(DEBUG),yes)
		@echo "Génération en mode debug"
	else
		@echo "Génération en mode release"
	endif

	hello: $(OBJ)
		@$(CC) -o $@ $^ $(LDFLAGS)

	main.o: hello.h

	%.o: %.c
		@$(CC) -o $@ -c $< $(CFLAGS)

	.PHONY: clean mrproper

	clean:
		@rm -rf *.o

	mrproper: clean
		@rm -rf $(EXEC)


Sous-makefiles
--------------

####Makefile MAITRE:####

	export CC=gcc
	export CFLAGS=-W -Wall -ansi -pedantic
	export LDFLAGS=
	HELLO_DIR=hello
	EXEC=$(HELLO_DIR)/hello

	all: $(EXEC)
	 
	$(EXEC):
		@(cd $(HELLO_DIR) && $(MAKE))

	.PHONY: clean mrproper $(EXEC)

	clean:
		@(cd $(HELLO_DIR) && $(MAKE) $@)

	mrproper: clean
		@(cd $(HELLO_DIR) && $(MAKE) $@)


####Makefile####

	EXEC=hello
	SRC= $(wildcard *.c)
	OBJ= $(SRC:.c=.o)

	all: $(EXEC)

	hello: $(OBJ)
		@$(CC) -o $@ $^ $(LDFLAGS)

	%.o: %.c
		@$(CC) -o $@ -c $< $(CFLAGS)

	.PHONY: clean mrproper

	clean:
		@rm -rf *.o

	mrproper: clean
		@rm -rf $(EXEC)
# Gérer les branches dans Git an 1'42

Les branches dans Git servent à séparer les différents sous-projets qu'un
projet peut avoir. Par exemple, si j'ai un programme qui marche déjà auquel je
veux ajouter une fonctionnalité, je peux créer une branche, développer mon code
dessus, le tester, l'adapter, puis une fois qu'il marche, merger la branche de
développement avec avec ma branche master. Sinon, on peut aussi fragmenter le
code par développeur, avoir chaque développeur qui a sa branche, et qui la
merge dans master périodiquement. Beaucoup de projets importants fonctionnent
avec une branche master sur laquelle se trouve un code déployable : on ne fait
pas de développement sur master, mais sur une autre branche (dev, à tout
hasard), et lorsqu'une nouvelle version est prête, on merge dev dans master.
C'est beau, c'est propre.

Concrètement la gestion des branches est super simple tant qu'on n'édite pas
les mêmes fichiers sur deux branches différentes. On crée une branche avec la
commande git branch <name> et on se déplace avec la commande git checkout
<name> (on peut utiliser git checkout -b <name> pour créer une branche et s'y
rendre directement). Pour incorporer dans la branche A les modifications faites
dans la branche B, on se place dans A et on lance git merge B

Concrètement, je suis dans mon projet :
(je suis sur master)

(modifications dans a.c et b.c)
        $ git commit -am "Fix bug in a.c and b.c."
        [master 2ef0361] Fix bug in a.c and b.c.
          2 files changed, 2 insertions(+), 0 deletions(-)
        $ git checkout -b dev
        Switched to a new branch 'dev'
(modifications dans b.c et c.c)
        $ git add b.c c.c
        $ git commit -m "Fix something in b.c and c.c."
        [dev 05ca674] Fix something in b.c and c.c.
          2 files changed, 2 insertions(+), 0 deletions(-)
        create mode 100644 c.c
        $ git checkout master
        Switched to branch 'master'
(modifications dans a.c)
        $ git commit -am "Add love and cuteness to a.c."
        [master 2115b7a] Add love and cuteness to a.c.
          1 files changed, 1 insertions(+), 0 deletions(-)

À l'heure actuelle, mon arbre git ressemble à ça :

          * 05ca674 2013-03-28: Patrick Lambein
          |     Fix something in b.c and c.c.
          |
        * | 2115b7a 2013-03-28: Patrick Lambein
        |/      Add love and cuteness to a.c.
        |
        * 2ef0361 2013-03-28: Patrick Lambein
        |       Fix bug in a.c and b.c.
        |
        * 88f4dbe 2013-03-28: Patrick Lambein
                Initial commit.
La branche de gauche (sur laquelle se trouve le commit 2115b7a) est la branche master, celle de droite est la branche dev.
Je merge mes branches :

        $ git merge dev
        Merge made by recursive.
         b.c |    1 +
         c.c |    1 +
         2 files changed, 2 insertions(+), 0 deletions(-)
         create mode 100644 c.c

Et maintenant mon arbre ressemble à ça :

        *   10298eb 2013-03-28: Patrick Lambein
        |\      Merge branch 'dev'
        | |
        | * 05ca674 2013-03-28: Patrick Lambein
        | |     Fix something something.
        | |
        * | 2115b7a 2013-03-28: Patrick Lambein
        |/      Add love and cuteness to a.c
        |
        * 2ef0361 2013-03-28: Patrick Lambein
        |       Fix bug in a.c and b.c
        |
        * 88f4dbe 2013-03-28: Patrick Lambein
                Initial commit

Un commit de merge a été créé, c'est pas très élégant, mais je n'ai pas eu de
problème.Mettons que je supprime ma branche dev (git branch -d dev). Je rajoute
un hello world.

        $ vim hello.c
        (…)
        $ git add hello.c
        [master 38e2949] Add hello.c.
          1 files changed, 8 insertions(+), 0 deletions(-)
        create mode 100644 hello.c

Je veux le modifier pour qu'il dise "Hello Patrick!" à la place. Je crée une
branche pour ça.

        $ git checkout -b hello_patrick
        Switched to a new branch 'hello_patrick'
        $ vim hello.c
        (…)
        $ git commit -m "Say Hello Patrick! instead."
        [hello_patrick 9c4ea27] Say Hello Patrick! instead.
          1 files changed, 1 insertions(+), 1 deletions(-)

Tout va pour le mieux dans le meilleur des mondes. Mais pendant que je
travaillais sur ma branche hello_patrick, Sacha a fait une modification au
fichier et lui fait dire "Hello Sacha!" Je retourne sur master et je fais un
git merge hello_patrick, et ça donne ça :

        $ git checkout master
        $ git merge hello_patrick
        Auto-merging hello.c
        CONFLICT (content): Merge conflict in hello.c
        Automatic merge failed; fix conflicts and then commit the result.

J'ai un merge conflict sur le fichier hello.c. Maintenant hello.c ressemble à ça :

        $ cat hello.c
        #include <stdio.h>

        int main(void)
        {
        <<<<<<< HEAD
                printf("Hello Sacha!\n");
        =======
                printf("Hello Patrick!\n");
        >>>>>>> hello_patrick

                return 0;
        }

Les passages du fichier en conflit sont affichés dans les deux versions. Il
faut que j'édite manuellement le fichier pour lui donner la forme que je veux,
et que je le commite ensuite.

        $ vim hello.c
        (…)
        $ git commit
        [master b78ad52] Merge branch 'hello_patrick'

(Je ne mets pas de -m parce que le message de commit a été généré par la
commande merge précédente)
Maintenant, mon arbre de commit ressemble à ça :

        *   b78ad52 2013-03-28: Patrick Lambein
        |\      Merge branch 'hello_patrick'
        | |
        | * 9c4ea27 2013-03-28: Patrick Lambein
        | |     Say Hello Patrick! instead.
        | |
        * | 46ca38e 2013-03-28: Sacha Delanoue
        |/      WIP
        |
        * 38e2949 2013-03-28: Patrick Lambein
        |       Add hello.c.
        |
        *   10298eb 2013-03-28: Patrick Lambein
        |\      Merge branch 'dev'
        | |
        | * 05ca674 2013-03-28: Patrick Lambein
        | |     Fix something something.
        | |
        * | 2115b7a 2013-03-28: Patrick Lambein
        |/      Add love and cuteness to a.c
        |
        * 2ef0361 2013-03-28: Patrick Lambein
        |       Fix bug in a.c and b.c
        |
        * 88f4dbe 2013-03-28: Patrick Lambein
                Initial commit

Ça a été plus pénible, mais j'ai bien fusionné mes branches après un merge.

Y'a encore des choses à dire, mais là j'en ai marre. Faites git help rebase si
vous êtes curieux.

En résumé :
git branch pour créer une branche
git checkout pour changer de branche
git merge pour fusionner des branches
Pour gérer les merge conflits, éditer les fichiers indiqués par git, faire un
git add de ces fichiers, puis git commit une fois que les conflits sont résolus.

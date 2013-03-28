# Faire des commits à peu près propres en 1'42

Git, c'est comme vim, c'est quelque chose qu'on devrait tous utiliser (par 
opposition à, par exemple, Mercurial et Emacs, ou Subversion et Microsoft Word). 
Du coup, ce post a pour but de rassembler les trucs, astuces, conseils et 
écueils à éviter à propos des commits dans Git.

Un commit n'est pas simplement une sauvegarde de votre code à un moment donné de 
votre progression, c'est également et surtout une étape sémantique. En lisant 
votre historique git (git log), vous voyez l'histoire de vos modifications 
apparaître, avec leur signification. Entre un commit et un autre, on a fait une 
modification qui a un sens (rajouté une fonction, factorisé du code, implémenté 
une fonctionnalité, etc.). Un code commité devrait au minimum compiler, 
idéalement être sans bug, et en pratique avoir l'air de faire ce que vous voulez 
qu'il fasse. Jusque là c'est pas compliqué, mais on s'embrouille vite, c'est 
pourquoi c'est hyper important de réfléchir à la nature du changement qu'on a 
fait depuis le dernier commit, et de pouvoir la résumer dans un message de 
commit. Si c'est difficile à résumer, c'est probablement qu'il fallait commiter 
avant.

Si par hasard on a fait plusieurs modifications qui ne sont pas liées entre 
elles, on gagne à les commiter séparément Exemple : sur la carte de TP, j'ai 
modifié un truc dans l'affichage du LCD et j'ai corrigé un bug dans la 
transmission XBee. Je vais effectuer les commandes suivantes :
        git add lcd.c lcd.h
        git commit -m "Add automatic return to line 2 if message length exceeds 
16 characters."
        git add xbee.c xbee.h
        git commit -m "Fix bug that transmitted a null character at the end of 
every string."

Du coup mon historique apparaît comme ça :

        git log

        commit c762b86f7c5edac65ef8d74dd6817cac7c4b0046
        Merge: c99d897 0daf33e
        Author: Patrick Lambein <patrick@lambein.name>
        Date:   Wed Mar 27 21:13:34 2013 +0100

                Fix bug that transmitted a null character at the end of every string.

        commit c99d897a4a31f0cf0e251c6050cd15f0d5208dfb
        Author: Patrick Lambein <patrick@lambein.name>
        Date:   Wed Mar 27 15:56:36 2013 +0100
        
                Add automatic return to line 2 if message length exceeds 16 characters.

        commit 0daf33e42f6903ce028b66342772993437435e53
        Author: John Evil <john@evilcorp.net>
        Date:   Wed Mar 27 11:31:00 2013 +0100
        
                Add bugs to fourier.c for the sake of pure evilness.

(Exemple fictif)

C'est bien, c'est clair, on voit bien ce que j'ai fait, et si j'ai fait 
n'importe quoi (comme 95% du temps), je peux assez rapidement identifier où ça 
s'est produit.

Si jamais vous avez fait un commit et que vous voulez y ajouter ou y modifier 
quelque chose, vous pouvez utiliser git commit --amend. Ça a le même effet que 
de faire un commit, mais ça rajoute les changements au dernier commit au lieu 
d'en créer un nouveau, et ça écrase le message de commit. ATTENTION : à 
n'utiliser que si l'on n'a pas encore pushé son code, un push rendant le commit 
définitif. Concrètement, supposons que je travaille à corriger un bug induit au 
commit 0daf33e42f6903ce028b66342772993437435e53, et qu'au milieu je suis 
interrompu par un livreur de sushis très impertinent qui m'apporte à manger. 
Je commite rapidement en exécutant :

        git add fourier.c
        git commit -m "WIP"

J'ai mangé mes sushis, je me suis remis à mon code, et maintenant je recommence 
à coder. J'ai fini de corriger mon bug, je peux donc amender mon commit :

        git add fourier.c
        git commit --amend -m "Fix bug to the function fourier_coef()."

        Mon historique donne maintenant :

        git log

        commit e2c229c1262ecb0b1692f4b0f2df443b7eb375c3
        Author: Patrick Lambein <patrick@lambein.name>
        Date:   Wed Mar 27 22:20:24 2013 +0100

                Fix bug to the function fourier_coef().

        commit c762b86f7c5edac65ef8d74dd6817cac7c4b0046
        Merge: c99d897 0daf33e
        Author: Patrick Lambein <patrick@lambein.name>
        Date:   Wed Mar 27 21:13:34 2013 +0100

                Fix bug that transmitted a null character at the end of every string.

        commit c99d897a4a31f0cf0e251c6050cd15f0d5208dfb
        Author: Patrick Lambein <patrick@lambein.name>
        Date:   Wed Mar 27 15:56:36 2013 +0100

                Add automatic return to line 2 if message length exceeds 16 characters.

        commit 0daf33e42f6903ce028b66342772993437435e53
        Author: John Evil <john@evilcorp.net>
        Date:   Wed Mar 27 11:31:00 2013 +0100

                Add bugs to fourier.c for the sake of pure evilness.

En gros, l'historique des commits ne devrait jamais avoir plus d'un WIP, et ça 
devrait toujours être votre dernier commit en date. Sinon on ne comprend pas ce 
que vous avez fait, et revenir sur son code devient rapidement galère. La règle 
que j'utilise est de ne pusher que du code qui marche et où je ne connais pas de 
bugs, et de garder le code sur mon ordinateur en attendant pour pouvoir amender 
mes commits avant d'avoir atteint ce stade.

# Les champs de bits en 1'42

Je ne suis probablement le seul en ROSE qui doit faire des manipulations bit à
bit, notamment pour une gestion fine de la mémoire. On l'a tous fait à un
moment ou à un autre pour régler des configurations, dans chibiOS par exemple.
Ça marche bien, mais dans le cas particulier où on utilise des struct, il
existe une autre façon de gérer des données bits par bits : les champs de bits
(bitfields en anglais).

Le principe est simple : si on veut déclarer un type struct pour coder, par
exemple, nos propres nombres flottants sur 32 bits (1 bit de signe, 8 bits
d'exposant et 23 bits de mantisse (exemple tiré d'ici) :

        struct my_float {
                unsigned int sign     : 1;
                unsigned int exponant : 8;
                unsigned int mantissa : 23;
        };

Ici, le champ sign fait effectivement 1 bit, le champ exposant en fait 8 et le
champ mantissa en fait 23, une donnée de type struct my_float en fait donc 32.

Concrètement donc, un champ se déclare comme
        <unsigned/signed> <type> <nom> : <taille>
Le comportement par défaut si l'attribut signed ou unsigned n'est pas utilisé
n'est pas partout spécifié, donc c'est une bonne habitude de le mettre. La
seule contrainte sur taille est de ne pas dépasser la taille maximum du type
type. Notez que l'attribut nom est facultatif, ce qui permet de faire du
padding, je reviendrai dessus. Je n'ai pas (encore) testé

La syntaxe de déclaration est très simple et est pratique pour une manipulation
sémantique plus claire des valeurs considérées, mais il faut faire très
attention, sous peine de pertes de performance en mémoire et/ou en exécution,
voire d'erreurs à la compilation.

On commence par le plus grave : les erreurs à la compilation. Un champ d'un bit
field n'a pas d'adresse, puisqu'il n'est pas forcément aligné sur une adresse
manipulable par la machine. Interdiction donc d'utiliser l'opérateur de
référencement &, ou le raccourci ->. Ça ne compilera pas, même si par hasard le
début de votre champ correspond avec le début d'un byte.

Ensuite pour les pertes de performances. J'utilise les bit fields pour réduire
la taille utilisée en mémoire, par exemple en utilisant un système d'adressage
personnel sur 11 bits. Dans mon struct, j'ai aussi des paramètres de types
normaux. Il faut faire attention à déclarer consécutivement les éléments d'un
bit field, sans quoi le résultat pourrait être contre-intuitif.

Exemple, j'ai le fichier suivant :

        struct my_struct {
                uint8_t param1 : 2;
                uint8_t param2;
                uint8_t param3 : 6;
        };

        struct other_struct {
                uint8_t param1 : 2;
                uint8_t param2 : 6;
                uint8_t param3;
        };

        int main(void)
        {
                printf("Taille de my_struct : %d\n", (int) sizeof(struct my_struct));
                printf("Taille de other_struct : %d\n", (int) sizeof(struct other_struct));
                return 0;
        }

À l'exécution, j'obtiens :

Taille de my_struct : 3
Taille de other_struct : 2

Explication : les éléments du premier bit field de my_struct ont été déclaré
comme faisant partie d'un uint8_t. Ils remplissent au total 2 bits. On demande
ensuite de déclarer un uint8_t. On considère alors que le bit field est
terminé, on passe au byte d'après, on place param2. On place ensuite les 6 bits
de param3 dans un nouveau bit field, et, puisqu'un struct doit occuper un
nombre entier de bytes, on prend aussi les deux derniers bits du troisième
byte. Dans le deuxième cas, on place dans un seul bit field param1 et param2,
et on place param3 dans le deuxième byte. Il faut donc faire attention de faire
des bit fields de tailles cohérentes, sans quoi on perd de la place.

Par ailleurs, on peut vouloir que certains champs commencent sur un nouveau
byte, pour des raisons d'efficacité dans la machine. Si je dois stocker des
paramètres sur 1, 6, 8, 4 et 2 bits, soit 21 bits au total, je perds de toute
façon 3 bits dans mon struct. Je peux donc, tant qu'à faire, le déclarer comme
ça :

        struct my_struct {
                uint8_t param1 : 1;
                uint8_t param2 : 6;
                uint8_t        : 1;

                uint8_t param3 : 8;

                uint8_t param4 : 4;
                uint8_t param5 : 2;
                uint8_t        : 2;
        };

L'avantage, c'est que pour accéder à param3, la machine ne devra manipuler
qu'un seul byte de la mémoire, contre deux si il avait été stocké directement
après param2. On optimise l'exécution du programme. J'ai explicitement défini
les deux bits finaux, c'est une bonne chose à faire, sans être nécessaire. Si
on déclare un champ d'une taille zéro, on force le prochain paramètre à
s'écrire sur le mot d'après. Je n'ai pas encore tout à fait finement compris
comment se passait l'alignement, en particulier si on mélange les types dans la
déclaration des champs. Beaucoup de ressources, dont la page de manuel des bit
fields, indiquent qu'il faut utiliser l'un des trois types int, signed int et
unsigned int, donc vous pourriez vouloir vous y restreindre.

Finalement, il faut savoir que les bit fields, étant très dépendant de la
machine, ajoutent souvent une grosse couche d'importabilité du programme. C'est
moins grave quand on travaille dans l'embarqué avec des applications écrites
pour des architectures très spécifiques, mais en général ce n'est pas forcément
une bonne pratique de s'en servir. Ceci dit, ça reste un outil pertinent pour
certains cas, et il m'est personnellement très utile, chaque octet économisé
dans mon struct correspondant à 1k d'économisé dans ma mémoire pas très grosse.

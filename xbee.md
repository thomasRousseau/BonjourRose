# En finir avec la confusion entre XBee et ZigBee en 1'42

J'ai remarqué qu'il y avait pas mal de confusions chez beaucoup de gens sur les 
protocoles radios qu'on utilise. Je vous propose un topo en une minute quarante 
deux secondes.

- Les radios que (je pense) tout le monde (sauf WaDeD pour une partie du projet) 
utilisent, celles qui se trouvent sur les cartes de TP, sont des modules XBee. 
C'est une marque de la firme Digi. Il existe plein de modèles de XBee différents 
qui implémentent différents protocoles réseaux. Le composant qu'on utilise est 
le XBee Pro 802.15.4, le "Pro" voulant dire "On a mis une radio plus puissante 
que dans le modèle normal".
- Le 802.15.4 est un protocole radio. Il est défini par l'IEEE. Il vise les 
communications à courte portée, de faible débit et à faible consommation (et 
coût). Il est proche de la couche matérielle, et propose peu de fonctionnalités 
réseau : les topologies offertes sont le point à point et l'étoile. C'est le 
protocole utilisé par les radios des cartes de TP, comme est subtilement indiqué 
par le nom du modèle.
- Le ZigBee est un protocole radio, développé par la ZigBee Alliance. C'est une 
surcouche du 802.15.4, qui offre des fonctionnalités réseau et applicatives. Le 
but du ZigBee est donc de faire du routage, d'utiliser des topologies 
sophistiquées et de communiquer entre applications. A priori, personne 
n'utilisera de ZigBee pour son projet, puisque personne n'a besoin de faire du 
réseau de façon sophistiquée.

Le problème vient du fait que XBee, ça ressemble assez fort à ZigBee. 
Il existe des XBee implémentant ZigBee, mais on n'en a pas. XBee est une marque 
de modules RF, ZigBee est un protocole. A priori, si vous hésitez entre les 
deux, vous n'avez aucune raison d'utiliser le mot "ZigBee".

En espérant que ça serve à quelqu'un.

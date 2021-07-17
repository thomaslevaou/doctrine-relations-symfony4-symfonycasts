# The Twig Extensions Library

Pour bien débuter notre page d'admin, on va lui faire afficher tous les commentaires qui existent 
en base.  

En utilisant le CommentRepository et quelques ajouts HTML/CSS/Twig mineur dans le template Twig,
on peut avoir facilement notre résultat.  
Mais on se retrouve alors avec 11 requêtes SQL exécutées dans le Profiler.  

Le problème c'est qu'à chaque itération de la boucle sur les commentaires, une requête
supplémentaire est effectuée. On se retrouve alors avec 11 requêtes : 1 qui liste tous les commentaires,
10 pour chacun des 10 articles associés à ces commentaires.
D'où le nom de **problème du N+1** : on a 10 objets associés à notre liste de commentaires + une requête 
pour obtenir la liste des commentaires.  

En fait, le fait de faire automatiser les requêtes par Doctrine peut entraîner la génération de 
beaucoup de requêtes SQL sur une page.  
Ce n'est pas toujours pertinent à corriger: en prod, avoir 10 requêtes sur une page d'admin n'est 
pas forcément gênant. Alors que 100 requêtes sur une page d'accueil, ça peut clairement plus l'être.  
D'où le conseil donné dans ce cours: "Livrez d'abord, voyez ensuite s'il y a des problèmes".  
Certains outils comme `Blackfire.io` permettent d'aider à repérer ce genre de problème si besoin.  

Pour le moment, on va d'abord voir comment limiter les tailles des commentaires à 30 caractères en Twig.
On a vu (même si je n'en ai plus trop le souvenir), que si on souhaitait ajouter des fonctions custom en Twig,
on pouvait utiliser le dossier `src/Twig`.  
Mais ici, on va installer une biblio qui s'appelle **Twig Extensions**. C'est juste une biblio open source 
ayant des fonctionnalités Twig supplémentaires.  

On l'installe via un `composer require twig/extensions --with-all-dependencies` : attention c'est une biblio PHP, pas un bundle Symfony,
donc l'ajout de service n'est pas fait "directement".

Mais cette biblio PHP a quand même installé une Recipe, ayant créé un fichier yaml `config/packages/twig_extensions.yaml`.  
On peut alors décommenter la ligne `Twig\Extensions\TextExtension: ~` (ou `null` à la place du `~`, qui dans les deux
cas veut dire que le service n'a pas besoin d'être configuré par un argument ou autre) du fichier.  
Et grâce aux paramètres en haut de ce fichier, Symfony remarque tout de suite que TextExtension est une extension Twig,
et en informe directement le service Twig.  
Concrètement ça veut dire qu'on peut appliquer notre nouveau filtre `|truncate` directement.  

La commande `./bin/console debug:twig` permet d'afficher la liste des outils associés à Twig dans notre appli.  
Dans la section des filtres, on peut bien voir la présence de `truncate`.


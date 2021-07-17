# Giving the Comments an isDeleted Flag

On cherche à présent à pouvoir effectuer une suppression logique de commentaires.

Pour ce faire, on commence par exécuter `php bin/console make:entity`.  
On va éditer la classe "Comment", en ajoutant une propriété `isDeleted` qui va être 
un booléen non nul. Puis comme d'hab : `./bin/console make:migration`.  

Dans le fichier de migration, notons que le type booléen est appliqué en tant que tinyint,
avec une valeur valant 0 ou 1.  Une fois cela fait, on migre :
`./bin/console doctrine:migrations:migrate`.  

Les `Comment` vont par défaut être mis à false dans l'entité
(`private $isDeleted = false;`).  
On précise dans la fixture que 20% des commentaires seront supprimés
logiquement : `$comment->setIsDeleted($this->faker->boolean(20));`
En refaisant un `./bin/console doctrine:fixtures:load`, on peut bien voir sur la page
certains commentaires marqués comme "supprimés" (après avoir ajouté un 
`<span class="fa fa-close"></span> deleted` dans le template Twig pour les commentaires 
supprimés).

Maintenant, on va chercher à ne pas afficher les commentaires supprimés. En soi, 
on pourrait faire directement une requête avec le queryBuilder de Doctrine,
où les commentaires ayant un "isDeleted" à false ne seraient pas sélectionnés.
Ou alors, une boucle sur tous les commentaires où on n'ajouterait que les
commentaires où "isDeleted" vaut false.

Ces deux solutions ne sont pas très optimisées en termes de performance. 
A la place, on va voir une meilleure solution dans le chapitre suivant.
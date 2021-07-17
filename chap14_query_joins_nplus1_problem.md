# Query Joins & Solving the N+1 Problem

Pour faire apparaître le mot recherché dans la barre de recherche après une 
recherche, on peut utiliser directement un raccourci Twig pour récupérer le 
paramètre GET de la requête : 
```HTML
<input type="text"
       name="q"
       class="form-control"
       placeholder="Search..."
       value="{{ app.request.query.get('q') }}"
>
```

En tapant la commande `./bin/console debug:twig`, on peut remarquer que la variable
`app`, que nous venons d'utiliser, est présente en tant que variable globale 
et est une instance de `Symfony\Bridge\Twig\AppVariable`.  

Sur PhpStorm, on peut retrouver la localisation d'un fichier via le raccourci
`Shift + Shift` (équivalent de `Ctrl + P` sur Visual Studio), ce qui permet de 
retrouver notre classe dans `/start/vendor/symfony/twig-bridge/AppVariable.php`.

Et dans cette classe, on peut effectivement voir la méthode `getRequest()` que 
Twig utilise ici pour accéder à la requête.  

Notons aussi qu'on peut retourner en Twig un message si aucun article n'a été 
trouvé, via un `{% else %}` dans la boucle for. C'est ce qu'on va faire sur 
notre page affichant les commentaires, pour indiquer qu'aucun commentaire n'a été 
trouvé dans notre liste.  

A présent, on souhaite faire en sorte que si on tape "bacon" dans la barre de
recherche, on ait aussi pour résultat les articles contenant "bacon" dans leur
titre. Pour ce faire, on va devoir utiliser une jointure (interne ici) : on rappelle que 
le titre de l'article est dans la table "Article", pas dans la table "Comment"
dans laquelle on a effectuée les recherches sur cette page jusqu'à présent.

Une jointure s'effectue avec Doctrine de la manière suivante : 
```PHP
$qb = $this->createQueryBuilder('c')
    ->innerJoin('c.article', 'a');
```
Le terme "article" utilisé ici fait référence à la propriété `article` de l'entité 
`Comment` (passée au constructeur du `CommentRepository`). L'article aura un 
alias "a" pour le reste de la requête.
Doctrine se charge ensuite de faire la jointure sur les id de comment et 
d'article, comme on peut le voir en jetant un oeil au profiler après avoir 
rechargé la page.  


On peut alors directement compléter la requête SQL :
```PHP 
if ($term) {
    $qb->andWhere('c.content LIKE :term OR c.authorName LIKE :term OR a.title LIKE :term')
        ->setParameter('term', '%'.$term.'%');
        
}
```

Dans l'appel à `innerJoin`, on va ajouter un appel à la fonction `addSelect()` pour 
sélectionner les éléments provenant à la fois de l'article et du commentaire lors 
de l'affichage des données :
```PHP
$qb = $this->createQueryBuilder('c')
    ->innerJoin('c.article', 'a')
    ->addSelect('a');
```

Ce qui permet d'avoir toutes les données d'un coup (en une seule requête),
et évite de devoir faire une requête supplémentaire pour avoir le titre
de chacun des articles associés à chaque commentaire.  
D'une manière générale, le `addSelect` permet souvent d'éviter des 
problèmes de performance et de requêtes parasites.  

Notons que même après cette jointure, la méthode `findAllWithSearch` 
retourne toujours un tableau de Commentaire (pas un tableau mixant
des commentaires et des Articles). Les données des Article sont juste
stockées en PHP ici pour être utilisées plus tard.
# Collection Magic with Criteria

Appeler une requête Doctrine ne serait pas forcément utile ici pour retourner
les commentaires supprimés (et surtout pas propre vu qu'on n'est pas dans le repo !):
on a déjà une méthode `getComments()` qui récupère 
tous les commentaires, alors autant en profiter. On pourrait faire une boucle sur 
`getComments` qui ajouterait à un nouveau tableau les commentaires non supprimés,
mais il existe un moyen plus optimisé via une utilisation de `Criteria`.  

Ca consiste concrètement à écrire notre méthode `getNonDeletedComments` de
l'entité `Article` de la manière suivante :
```PHP
public function getNonDeletedComments(): Collection
{
    $criteria = Criteria::create()
        ->andWhere(Criteria::expr()->eq('isDeleted', false))
        ->orderBy(['createdAt' => 'DESC']);

    return $this->comments->matching($criteria);
}
```

Et lorsqu'on exécute la page, on voit bien qu'une requête a été créée pour 
récupérer des commentaires, sans avoir eu besoin d'utiliser un querybuilder
de notre côté.

Le Criteria est pratique quand on doit récupérer une petite partie d'élements 
d'une liste. Il est plus optimisé qu'une boucle for qui ferait le tri sur 
les `$comments`.  

Histoire de garder les logiques de requête dans le repository, on ajoute 
le Criteria dans le repository :
```PHP
public static function createNonDeletedCriteria(): Criteria
{
    return Criteria::create()
        ->andWhere(Criteria::expr()->eq('isDeleted', false))
        ->orderBy(['createdAt' => 'DESC']);
}
```

On utilise une fonction statique pour pouvoir l'appeler facilement dans l'Article :
```PHP
public function getNonDeletedComments(): Collection
{
    $criteria = ArticleRepository::createNonDeletedCriteria();
    return $this->comments->matching($criteria);
}
```

Notons qu'on aurait aussi pu mettre la méthode statique dans `CommentRepository`, ça dépend des besoins de l'appli
et c'est rapide à déplacer de toute façon.

Pour info, le criteria est aussi utilisable dans le querybuilder, via la méthode `addCriteria(self::createNonDeletedCriteria())` ici.
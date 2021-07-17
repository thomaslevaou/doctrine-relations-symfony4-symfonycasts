# ManyToMany Joins & When to Avoid ManyToMany

De la même manière que lors du problème du N+1 avec les ManyToOne / OneToMany,
on va résoudre le problème de performance vu à la fin du chapitre précédent par un 
`join` fait en amont.

On va ajouter ce join dans la méthode `findAllPublishedOrderedByNewest()` de 
`ArticleRepository` (c'est la méthode appelée dans la méthode `homepage` de 
`ArticleController`).  
En SQL, on aurait du devoir faire deux jointures internes : une vers la table 
`article_tag`, puis une vers la table `tag` avec l'id de tag remonté par la
table d'association. Mais en Doctrine, il nous suffit d'appliquer la méthode 
`leftJoin()` avec l'alias `t` :
```PHP
public function findAllPublishedOrderedByNewest(): array
    {
        return $this->addIsPublishedQueryBuilder()
            ->leftJoin('a.tags', 't')
            ->addSelect('t')
            ->orderBy('a.publishedAt', 'DESC')
            ->getQuery()
            ->getResult()
            ;
    }
```

Le `tags` dans `a.tags` fait toujours référence à la propriété `tags` 
de l'entité `Article`. Notons qu'ici faire un left ou un inner join importe 
peu (il n'y a pas de tag sans article).

Par contre le souci si on laisse Doctrine tout gérer avec la table 
d'association, c'est qu'on ne peut pas utiliser la table d'association
pour enregistrer des données spécifiques à une association entre 
un article et un tag. Par exemple, là comme ça on ne peut 
pas enregistrer la date à laquelle un article a été associé à un
tag donné.  

Le simple fait d'utiliser une relation ManyToMany empêche 
d'avoir des données fonctionnelles dans une table d'association.  

A la place, dans la situation d'exemple on devra créer nous-mêmes
l'entité `ArticleTag`, avec une relation `ManyToOne` pour l'article,
et une relation `ManyToOne` pour le tag, et les données d'association
qu'on veut ensuite.  
Une migration des données présentes dans la table d'association Doctrine
sera toujours possible en cas de mauvais choix. Mais mieux vaut prendre 
les bonnes décisions dès le début pour gagner du temps ou de la performance.
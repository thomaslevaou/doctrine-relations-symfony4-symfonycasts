# OrderBy & fetch EXTRA_LAZY

On a des commentaires auto-générés sur les articles. C'est cool, mais là ils sont
affichés dans n'importe quel ordre (pas du plus récent au plus vieux comme on aurait
souhaité).
Pour pallier à ce problème, on va ajouter une nouvelle annotation `OrderBy` dans 
l'entité de l'Article :
```PHP
/**
 * @ORM\OneToMany(targetEntity=Comment::class, mappedBy="article")
 * @ORM\OrderBy({"createdAt" = "DESC"})
 */
private $comments;
```
Et comme ça dans sa requête de prélèvement des commentaires, Doctrine va
appliquer l'ordre désiré ici.  
Notons que l'annotation ne fonctionne qu'avec des guillemets : si on met 
des apostrophes à la place, elle n'est pas comprise correctement.

Pour avoir le nombre de commentaires sur chaque article quand on voit 
la liste sur la page d'accueil, il suffirait en soit d'appliquer un 
`<small>({{ article.comments|length }} comments)</small>` dans 
`homepage.html.twig`. Mais le problème est que cette méthode n'est clairement 
pas optimisée :  dans le profiler, on peut voir que ce simple appel 
de commentaires a entraîné six nouvelles requêtes, pour sélectionner
tous les commentaires et les compter ensuite, et ce pour chacun des articles.

Pour pallier à ce problème, on ajoute un `fetch="EXTRA_LAZY"` dans l'annotation
`OneToMany` des commentaires de l'entité de l'article :
```PHP
/**
 * @ORM\OneToMany(targetEntity=Comment::class, mappedBy="article", fetch="EXTRA_LAZY")
 * @ORM\OrderBy({"createdAt" = "DESC"})
 */
private $comments;
```

Ainsi, Doctrine fera juste un `count(*)` directement pour récupérer le nombre de 
commentaires de chaque article lorsqu'on aura besoin d'appeler `comments|length`.

Si le `fetch="EXTRA_LAZY"` n'est pas toujours employé, c'est parce que dans certains
cas il n'est pas bon pour la performance : sur les détails d'un article, où le nombre
de commentaires est affiché en haut et les contenus des commentaires plus bas, un
`count()` est appelé pour compter les commentaires. Alors qu'avant le extra_lazy,
Doctrine récupérait les contenus des commentaires et en déduisait tout seul le
`count()`.

D'où le fait que les optimisations de performance dépendent des situations : du coup, 
d'abord on développe, ensuite on voit s'il y a des soucis de perf, et ensuite on les 
corrige. Mettre un extra_lazy systématiquement n'est pas toujours bon.
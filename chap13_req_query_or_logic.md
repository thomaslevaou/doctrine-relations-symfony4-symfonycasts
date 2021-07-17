# Request Object & Query OR Logic

Sur la [page des commentaires](http://localhost:8000/admin/comment), on va ajouter 
un champ de recherche pour filtrer les commentaires.  

Pour ce faire, on ajoute un formulaire dans `templates/comment_admin/index.html.twig` : 
```HTML
<form>
    <div class="input-group mb-3">
        <input type="text"
               name="q"
               class="form-control"
               placeholder="Search..."
        >
        <div class="input-group-append">
            <button type="submit"
                    class="btn btn-outline-secondary">
                <span class="fa fa-search"></span>
            </button>
        </div>
    </div>
</form>
```

On rappelle que le fait de ne pas préciser l'attribut `method` de la balise 
`<form>` indique que le contenu du formulaire va être envoyé en méthode GET (et 
c'est ce qu'on veut ici).  

Comme vu à Label Emmaüs, pour lire des contenus GET, POST, des headers, cookies,
fichiers, etc, on va
utiliser une instance de la classe Symfony `Request` de `HttpFoundation`.  
On va l'ajouter dans `CommentAdminController`, et l'utiliser de la manière suivante :
```PHP
public function index(CommentRepository $repository, Request $request): Response
{
    $q = $request->query->get('q');
    ...
}
```
Attention, la classe Request n'est pas un Service (bien qu'on pourrait 
utiliser `RequestStack` si vraiment on veut un service). Mais ça n'a pas
d'importance en pratique: dans une méthode d'un contrôleur, la classe Request est
quand même reconnue automatiquement (on peut la type-hint). C'est le troisième "pouvoir" des 
contrôleurs après l'auto-wiring des Services et la génération de requêtes par 
annotation pour les entities.  

D'une manère générale, en Symfony les commandes `$_GET`, `$_POST`, ou `$_SERVER`
sont à proscrire au profit d'utilisation de la classe Symfony `Request`.  

Par contre, pour faire notre requête pour récupérer les données en base, on ne peut 
pas utiliser `findBy` pour faire des LIKE.  

A la place, on va utiliser une nouvelle méthode `findAllWithSearch()` dans 
`CommentRepository` :
```PHP
/**
     * @param string|null $term
     * @return Comment[]
     */
    public function findAllWithSearch(?string $term)
    {
        $qb = $this->createQueryBuilder('c');

        if ($term) {
            $qb->andWhere('c.content LIKE :term OR c.authorName LIKE :term')
                ->setParameter('term', '%'.$term.'%');
        }

        return $qb->orderBy('c.createdAt', 'DESC')->getQuery()->getResult();
    }
```

Le point d'interrogation avant le type du paramètre (`?string`) est là pour dire que
ce paramètre peut être null (donc facultatif).  
Les annotations `@param` et `@return` sont là pour permettre une génération
automatique de documentation.  
Notons l'existence de `orWhere` qui nous permet de recherche dans deux tables en 
même temps, mais elle est relou à utiliser (trop parenthèse-phile). Il vaut 
mieux utiliser `andWhere` avec un `OR` dedans.
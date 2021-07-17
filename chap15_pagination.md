# Pagination

On va ajouter une pagination pour mieux afficher nos commentaires.  
Pour ce faire, on va utiliser ici le `KnpPaginatorBundle`.  
On l'installe via `composer require knplabs/knp-paginator-bundle`.  

Grâce à Flex, la config d'installation est faite automatiquement en 
Symfony 4.  

La doc du KnpPaginatorBundle sur le GitHub n'est plus à jour (en fait 
si mais on va faire comme si la doc ne s'était pas mise à jour, 
pour s'entraîner). En effet, la récupération d'un service est faite 
ici via un `$this->get('knp_paginator')`, ce qui n'est plus vraiment 
possible en Symfony 4. Pour résoudre le souci, on va recherche le
Service associé à cet identifiant via la commande
`./bin/console debug:autowiring`, ce qui permet de retrouver le 
service qu'on souhaite, à savoir `PaginatorInterface`.  

Comme une instance de PaginatorInterface demande un objet du type
`QueryBuilder`, on va modifier le code de notre fonction
`findAllWithSearch` dans notre `CommentRepository` pour récupérer 
un QueryBuilder à la place d'un tableau de commentaires :
```PHP
/**
 * @param string|null $term
 */
public function getWithSearchQueryBuilder(?string $term): QueryBuilder
{
    $qb = $this->createQueryBuilder('c')
        ->innerJoin('c.article', 'a')
        ->addSelect('a');

    if ($term) {
        $qb->andWhere('c.content LIKE :term OR c.authorName LIKE :term OR a.title LIKE :term')
            ->setParameter('term', '%'.$term.'%');
    }

    return $qb->orderBy('c.createdAt', 'DESC');
}
```
On a renommé notre méthode en `getWithSearchQueryBuilder`,
retiré l'annotation de retour (`@return Comment[]`), ajouté un `QueryBuilder`
en type de retour, et retiré le `getQuery()->getResult()` dans le `return` de la
dernière ligne de la méthode.  


Notre méthode `index` de `CommentAdminController` appelle maintenant notre 
pagination, et devient de ce fait la suivante :
```PHP
/**
* @Route("/admin/comment", name="comment_admin")
*/
public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator): Response
{
    $q = $request->query->get('q');

    $queryBuilder = $repository->getWithSearchQueryBuilder($q);

    $pagination = $paginator->paginate(
      $queryBuilder,
      $request->query->getInt('page', 1),
      10
    );

    return $this->render('comment_admin/index.html.twig', [
        'pagination' => $pagination,
    ]);
}
```

Pour appeler la pagination dans le template Twig, on va utiliser quelques
raccourcis que le template offre: 
- `{{ pagination.getTotalItemCount }}` pour le nombre total de commentaires 
retournés par la requête;
- `{{ knp_pagination_render(pagination) }}` pour afficher immédiatement un 
rendu visuel de pagination.
  
Notons que la pagination prend en compte les critères de recherche ! 

Pour paramétrer notre style de pagination (et avoir une pagination
un minimum présentable), on va créer ici notre propre
fichier de config `config/packages/knp_paginator.yaml` et lui ajouter
le contenu suivant (conformément aux indications de la doc du bundle): 
```YAML
knp_paginator:
  template:
    pagination: '@KnpPaginator/Pagination/twitter_bootstrap_v4_pagination.html.twig'
```
Et comme ça on a une pagination joliment présentable sur la page.
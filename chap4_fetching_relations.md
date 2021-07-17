# Fetching Relations

Maintenant qu'on a inséré des commentaires en base, il serait temps de les afficher
sur la page.  
Le MakerBundle a déjà généré pour nous le `CommentRepository` associé au Comment,
qu'on va pouvoir utiliser. Il s'agit toujours d'un service, on peut donc l'appeler 
dans la fonction `show()` de `ArticleController`, et récupérer les commentaires 
de l'article via un 
`$comments = $commentRepository->findBy(['article' => $article]);`.  
Mais dans notre contexte, le meilleur moyen reste de faire 
`$comments = $article->getComments();` (qui retourne juste la collection, mais 
qui saura appeler automatiquement les commentaires associés au moment opportun).

Et en fait, on pourra appeler les commentaires directement en Twig. Du coup, 
on va carrément supprimer le code appelant les commentaires en Symfony, ce qui va 
beaucoup alléger les commentaires dans la méthode `show()` : 
```PHP
public function show(Article $article, SlackClient $slack)
{
    if ($article->getSlug() === 'khaaaaaan') {
        $slack->sendMessage('Kahn', 'Ah, Kirk, my old friend...');
    }

    return $this->render('article/show.html.twig', [
        'article' => $article
    ]);
}
```

Et on peut simplement boucler sur les comments dans les commentaires dans 
le fichier Twig : 
```HTML
{% for comment in article.comments %}
<div class="row">
    <div class="col-sm-12">
        <img class="comment-img rounded-circle" src="{{ asset('images/alien-profile.png') }}">
        <div class="comment-container d-inline-block pl-3 align-top">
            <span class="commenter-name">{{ comment.authorName }}</span>
            <small>about {{ comment.createdAt|date('Y-m-d') }}</small>
            <br>
            <span class="comment"> {{ comment.content }}</span>
            <p><a href="#">Reply</a></p>
        </div>
    </div>
</div>
{% endfor %}
```

On peut bien voir dans le profiler les deux requêtes qui ont été faites :
celle pour récupérer l'article, et celle pour récupérer les commentaires.

La deuxième requête a été appelée uniquement au moment d'appeler les commentaires 
dans le code Twig: c'est ce qu'on appelle la **fainéantise de Doctrine**
(Doctrine laziness) qui est pratique pour éviter d'exécuter des requêtes inutilement.
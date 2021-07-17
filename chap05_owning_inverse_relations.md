# Owning vs Inverse Relations

On a vu qu'on pouvait obtenir les commentaires d'un article via `getComments()`, ou 
alors obtenir l'article associé à un commentaire via `getArticle()`.

Si on remplace les `$comment->setArticle($article);` par des
`$article->addComment($comment)`, et qu'on reload les fixtures (`./bin/console 
doctrine:fixtures:load`), on peut voir que les commentaires sont insérés en base aussi.  
 
Si on regarde dans le code de la méthode `addComment()`, on peut voir que le commentaire
est ajouté "des deux côtés" de la relation : concaténé dans le `comments[]` de
l'article, et associé via un `setArticle()` dans le commentaire.  

Le côté de la relation étant en `ManyToOne` est appelé le **owning side**.  
L'autre côté, en `OneToMany`, est appelé l'**inverse side**.  

Là où le côté owning est obligatoire, le côté inverse est facultatif: pour rappel,
la commande nous a demandé si on souhaitait un inverse side lors de la création de la 
relation. L'inverse side ne sert qu'à avoir facilement un `getComments()`, permis 
grâce à cette relation OneToMany.
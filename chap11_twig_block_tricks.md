# Twig Block Tricks

On va créer une page d'administration sur le site.  
On commence par créer un nouveau controller, ce qu'on va faire via la commande 
`./bin/console make:controller`, en l'appelant `CommentAdminController`.  
Cette commande a non seulement créé notre fichier controller, mais aussi un template 
Twig associé dans un nouveau dossier `comment_admin`.

En regardant le fichier `comment_admin/index.html.twig`, on peut voir que notre nouveau template 
hérite déjà du template de base.  

On peut faire des components en Twig, mais ça je le sais déjà parce que j'ai déjà vu à Label.

Pour ajouter des classes uniquement à certains appels de components, on va utiliser un nouveau
block, qu'on va appeler `content_class` et appliquer tel que ci-dessous.

Dans `content_base.html.twig`:
```HTML
<div class="container">
    <div class="row">
        <div class="col-sm-12">
            <div class="{% block content_class %}show-article-container p-3 mt-4{% endblock %}">
                {% block content_body %}
                {% endblock %}
            </div>
        </div>
    </div>
</div> 
```

Dans `comment_admin` :
```HTML
{% block content_class %}{{ parent() }} show-article-container-border-green{% endblock %}
```

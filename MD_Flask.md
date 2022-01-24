# Flask

Flask est un petit framework web Python léger, qui fournit des outils et des fonctionnalités utiles qui facilitent la création d'applications web en Python.

Flask utilise le moteur de modèle Jinja pour construire dynamiquement des pages HTML en utilisant des concepts Python familiers tels que les variables, les boucles, les listes, etc.

[un comparatif Flask/Django](https://www.kicklox.com/flask-vs-django-framework-python/#:~:text=Flask%20est%20aujourd'hui%20tr%C3%A8s,une%20architecture%20en%20micro%2Dservices.&text=Django%20est%20un%20framework%20de,suivant%20une%20architecture%20bien%20pr%C3%A9cise.)

Il faut installer Flask avec la commande `pip` : `pip install flask`

    `#import de la biblio
    from flask import Flask`

## le serveur web

Voici comment démarrer un serveur web et afficher une première information sur un navigateur.

    `# __name__ super variable qui contient le nom du module Python courant
    app = Flask(__name__)

    # décorateur qui va servir la fonction en réponse http
    # ici c'est la racine du serveur web '/'
    @app.route('/')
    def hello():
        return 'Hello, World!'

    # Démarer le serveur
    # le debug=true permet d'afficher les messages du serveur dans la console
    if __name__ == "__main__":
        app.run(debug=True)`

## page html

Les applications web utilisent principalement le HTML pour afficher des informations. Il faut donc intégrer des fichiers HTML à votre application, pour obtenir des pages web.

Flask fournit une fonction d'aide `render_template()` qui permet l'utilisation du [moteur de modèle Jinja](https://jinja.palletsprojects.com/en/2.11.x/). Cela facilitera grandement la gestion des pages en écrivant votre code directement dans des fichiers `.html`.

    `from flask import Flask, render_template

    app = Flask(__name__)

    @app.route('/')
    def index():
        # return le rendu d'un fichier html
        return render_template('index.html')`

Pour que ça fonctionne, le fichier `index.html` doit être dans un répertoire `templates` placé à la racine du projet.

Les applications web de Flask disposent aussi d'un dossier `static` pour l'hébergement des fichiers ressources (css, images, js...).

    `<!-- fichier templates/index.html -->
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Python web</title>
        <!-- va rechercher les ressources dans /static grâce à url_for()-->
        <link rel="stylesheet" href="{{ url_for('static', filename= 'css/style.css') }}">
    </head>
    <body>
        <h1>Hello</h1>
    </body>
    </html>`

l'injection de code, dans le template, se fait par `{{}}` et la méthode `url_for()` va générer le chemin à partir du répertoire `static` lorsque celui-ci est spécifié en premier argument, et dont le fichier est défini en second argument `filename= 'css/style.css'`.

    `/* fichier static/css/style.css */
    h1{
        width:100%;
        color:white;
        background-color: black;
        text-align: center;
    }`

Ce code a pour résultat :

![Page web]()

## L'héritage de modèles

On va profiter du code Python pour ne pas avoir toutes les pages à créer en HTML. [Jinja propose un système d'héritage](https://jinja.palletsprojects.com/en/2.10.x/templates/#template-inheritance) qui aide à la conception de modèles et à sa réutilisation dans les pages web.

`<!-- fichier templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %} {% endblock %}</title>
    <!-- va rechercher les ressources dans /static grâce à url_for()-->
    <link rel="stylesheet" href="{{ url_for('static', filename= 'css/style.css') }}">
</head>
<body>
    <article>{% block content %} {% endblock %}</article>
    <a href="{{ url_for('index') }}">retour à la page d'accueil</a>
</body>
</html>`

- {% block title %} {% endblock %}: ce bloque de titre indique où il faut insérer le titre sur la page html.
- {{url_for('index')}}: retourne l'url correspondant à la fonction index()dans le code Python.
- {% block content %} {% endblock %}: ce bloque de contenu indique où il faut insérer le contenu sur la page html.

Il existe encore beaucoup d'autres bloques, détaillés dans la doc officiel de Jinja.

    `<!-- fichier templates/index.html -->
    {% extends 'base.html' %}


    {% block content %}
        <h1>{% block title %} Hello {% endblock %}</h1>

        <p>
        Lorem ipsum dolor sit, amet consectetur adipisicing elit. Labore sit, autem aspernatur numquam inventore ut illum excepturi laborum magnam vitae quaerat distinctio assumenda ea. Perspiciatis facilis exercitationem optio laudantium quis.
        </p>
    {% endblock %}`

##Data binding

Il est possible de passer des valeurs au fichier html à partir du Python en passant par un dictionnaire.

    `#import de la biblio
    from flask import Flask, render_template

    # __name__ super variable qui contient le nom du module Python courant
    app = Flask(__name__)

    # décorateur qui va servir la fonction en réponse http
    # ici c'est la racine du serveur web '/'
    @app.route('/')
    def index():
        menus = ["à gauche", "à droite"]
        # dictionnaire de valeurs à paser à la page web
        contenu = {"titre":"Hello", "menu":menus}
        # render_template() prend en paramètre le dictionnaire de valeurs
        return render_template('index.html', contenu = contenu)

    if __name__ == "__main__":
        app.run(debug=True)`

    `<!-- fichier templates/index.html -->

    {% extends 'base.html' %}


    {% block content %}
        <h1>{% block title %} {{contenu["titre"] }} {% endblock %}</h1>

        <p>
        Lorem ipsum dolor sit, amet consectetur adipisicing elit. Labore sit, autem aspernatur numquam inventore ut illum excepturi laborum magnam vitae quaerat distinctio assumenda ea. Perspiciatis facilis exercitationem optio laudantium quis.
        </p>
        <ul>
            {% for el in contenu["menu"] %}
            <li>{{el}}</li>
            {% endfor%}
        </ul>
    {% endblock %}`

L'injection de valeurs se fait par `{{}}`. Il faut reprendre le nom de la variable donnée dans le `render_template()` du Python, et préciser entre `[]` la clé de la variable à injecter, puisque c'est un dictionnaire. A noter, qu'il est parfaitement possible de parcourir une liste avec un for dans le template `{% for el in contenu["menu"] %}`.

## URL dynamique

Il est possible de passer des informations dans les URL grâce à la [règle de variable](https://flask.palletsprojects.com/en/1.1.x/quickstart/#variable-rules).

    `# web.py 
    #url dynamique
    @ap.route('/afficher/<string:mot>')
    def afficher(mot):
        # grâce au paramètre, qui porte le même nom que l'url dynamique,cette partie est récupérée dans la fonction
        # dans cette exemple, elle est passé en paramètre à la page web
        return render_template('afficher.html', mot = mot)`

    `<!-- fichier templates/afficher.html -->
    {% block content %}
        <!-- récupère  sous forme de variable la partie dynamique de l'URL -->
        <h1>{% block title %} {{mot}} {% endblock %}</h1>

        <p>
            {{mot}}
        </p>
    {% endblock %}`

Il ne reste plus qu'à générer l'URL dynamique, dans une page web, en passant un nouvel argument à `url_for()`.

    `<!-- fichier templates/base.html -->
    <a href="{{ url_for('afficher', mot='gorille')}}">Afficher mot</a`

## Documentation

- [la doc officielle](https://flask.palletsprojects.com/en/1.1.x/)
- [Tuto VS Code](https://code.visualstudio.com/docs/python/tutorial-flask)
- [TP Flask](file:C:/Users/Franky TANGUY/Code/Python/Flask2/kata_flask.pdf)

## Déploiement

Flask utilise un simple serveur web pour servir une application dans un environnement de développement, ce qui signifie également que le débogueur Flask est en cours d'exécution pour faciliter la capture des erreurs. Ce serveur de développement ne doit pas être utilisé dans un déploiement de production. Consultez la page des [options de déploiement](https://flask.palletsprojects.com/en/1.1.x/deploying/) dans la documentation Flask pour plus d'informations, vous pouvez également consulter ce tutoriel sur [le déploiement de Flask](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04).

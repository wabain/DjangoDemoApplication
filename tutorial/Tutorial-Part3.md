# Writing Our Application

## Models

You typically start development by writing the database models. These will identify and store the data that we will use to populate our website with content.

Start by creating three files in your `models` directory, one for each model we will write: `snippet.py`, `tag.py`, and `person.py`. **You should always use the singular form for naming models**.

Let's start with the Snippet model. In `snippet.py` add the following code. Make sure you read it and understand what's going on before copying and pasting.

`models/snippet.py`:

    from django.db import models


    class Snippet(models.Model):
        class Meta:
            app_label = "codekeeper"

        title = models.CharField(max_length=256, blank=True, null=True)
        snippet = models.TextField()
        tags = models.ManyToManyField("codekeeper.Tag", blank=True, null=True)
        creator = models.ForeignKey("codekeeper.Person")

        created = models.DateTimeField(auto_now_add=True)
        updated = models.DateTimeField(auto_now=True)

        def __str__(self):
            return "{0}".format(self.title)

If you are familiar with Django models, this looks pretty straightforward. There may just be a few things that you are not aware of. The first is the Meta-class `app_label` attribute. When we break out our models into individual files, this is a necessary attribute that helps our application loader find a given model.

The `__str__` method determines what field Django uses to describe each model instance to the user. We've chosen the `title` field. This will make it easier to identify each model in the Django admin interface.

> In Python 2.7, the `__unicode__` method is used in place of `__str__`. This is because 2.7
> distinguishes between Unicode and ASCII, while 3+ uses Unicode for everything.

We will do the same thing for Tag and Person now.

`models/person.py:`

    from django.db import models


    class Person(models.Model):
        class Meta:
            app_label = "codekeeper"

        first_name = models.CharField(max_length=256, blank=True, null=True)
        last_name = models.CharField(max_length=256, blank=True, null=True)

        created = models.DateTimeField(auto_now_add=True)
        updated = models.DateTimeField(auto_now=True)

        def __str__(self):
            return "{0}, {1}".format(self.last_name, self.first_name)

`models/tag.py`:

    from django.db import models


    class Tag(models.Model):
        class Meta:
            app_label = "codekeeper"

        name = models.CharField(max_length=255)
        created = models.DateTimeField(auto_now_add=True)
        updated = models.DateTimeField(auto_now=True)

        def __str__(self):
            return "{0}".format(self.name)

Notice that we are using Foreign Key fields to relate each instance to another model. In the `Snippet` model we point to the `Person` and `Tag` objects that store data about that particular person and a reference to a list of tags.

One last thing we need to do is to add a reference to each of these in the `__init__.py` file in our `models` directory. Open up this file and add:

```
from codekeeper.models.snippet import Snippet
from codekeeper.models.person import Person
from codekeeper.models.tag import Tag
```

This important step allows Django to pick up on these models in the database synchronization system.

## Serializers

We will need at least one serializer for every model we create. In the `serializers` folder you created earlier, create three new files named the same as the models: `snippet.py`, `person.py` and `tag.py`. These will be pretty simple to start with.

`serializers/snippet.py`

    from rest_framework import serializers
    from codekeeper.models.snippet import Snippet


    class SnippetSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = Snippet

`serializers/person.py`

    from rest_framework import serializers
    from codekeeper.models.person import Person


    class PersonSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = Person

`serializers/tag.py`

    from rest_framework import serializers
    from codekeeper.models.tag import Tag


    class TagSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = Tag

## Views

Next, let's create a couple basic views so that we can work with our system. In your `views` folder create a file, home.py.

`views/home.py`:

    from rest_framework import views
        from rest_framework import renderers
        from rest_framework.response import Response
        from rest_framework.reverse import reverse
        from codekeeper.renderers.custom_html_renderer import CustomHTMLRenderer


        class HomePageView(views.APIView):
                template_name = "index.html"
            renderer_classes = (CustomHTMLRenderer,
                        renderers.JSONRenderer,
                        renderers.BrowsableAPIRenderer)

                def get(self, request, *args, **kwargs):
                    response = Response()
                    return response


This sets up our homepage view, which we will eventually hook up in our `urls.py`.

## Templates

For now, let's use a very simple HTML5 template. Create a new file in your `templates` directory called "base.html". In it put the following code:

```
{% load staticfiles %}

<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Code Keeper Snippets</title>

  <link rel="stylesheet" href="{% static 'css/styles.css' %}">

  <!--[if lt IE 9]>
  <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
  <![endif]-->
  <script src="{% static 'js/scripts.js' %}"></script>
</head>

<body>
{% block body %}

{% endblock %}
</body>
</html>
```

While we're at it, let's create some folders to hold our static JavaScript and CSS assets. Create two folders in the `static` directory, `css` and `js`. Since we reference files in these directories in our template, create `styles.css` in the css directory, and `scripts.js` in the js directory. We can leave them blank for now.

Let's now create the template snippet for our website's front page. Create a new file, `index.html` in your templates directory. In it place the following:

```
{% extends "base.html" %}

{% block body %}
    <h1>Hello World</h1>
{% endblock %}
```

This will be a temporary front page for our website.

# Connect the views

Now, let's connect the view we created and map it to a URL we can visit in a web browser. Open up the `urls.py` file and change it to this:

`urls.py`:

    from django.conf.urls import patterns, include, url
    from django.contrib import admin

    from codekeeper.views.home import HomePageView

    urlpatterns = patterns('',
        url(r'^$', HomePageView.as_view(), name="home"),
        url(r'^admin/', include(admin.site.urls)),
    )

Note in particular the two lines that reference our view: The first when we import it from our views folder, and the second where we connect it to the root of our website. The regular expression `r'^$'` indicates the base URL and `HomePageView.as_view()` is the code that handles it.

# Running our application

## Configure and Initialize the database

Django has a number of convenient functions for managing databases and keeping it in sync with the code we write. Our "models" that we write will be automatically turned into database tables used for storing the data, and match the structure of the fields we describe in our models.

Before starting our application we must first synchronize our database. This process converts the models that we just wrote into the database structure.

First, we need to indicate which database engine we are using. By far, the easiest to develop with is SQLite. This will create a single database file in your project directory and allow you to get up and running quickly.

Open your `settings.py` file and look for the `DATABASES` section. Change it to the following:

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'codekeeper.sqlite3'),
        }
    }

This will create a file, `codekeeper.sqlite3` in our project directory.

## Database Migrations

A migration is a method of updating a database without needing to wipe and re-create a database from scratch if we change our models. Remember that our models are directly tied to the database structure, so if we edit the code for our models—for example, adding or removing a field—we must also make sure the data contained in the database reflects these edits.

Without migrations, updating our website with existing data is an awkward and error-prone process. If we wanted to make a change to our models in a production website we would need to dump the data, re-structure it according to a new structure, and then re-import it into the new database structure. This is a lot of work, and can lead to loss of data if you're not careful.

Migrations keep track of your model changes and helps synchronize your database without dumping your data. (You should have a backup on hand, though, in case it fails!)

To begin, we must first describe the initial state of our database models.

    $> python manage.py makemigrations codekeeper

This tells Django to create an initial migration for our application.

If successful you should now see a 'migrations' folder in your project. As you change your models you will run a similar command and the changes to the database structure will be kept in Python files in this folder.

For now, however, let's continue with getting our database set up.

## Synchronizing our database (the first time)

Synchronizing the database converts the Python models to actual database tables and fields. To synchronize your database, run the following command:

    $> python manage.py syncdb

If this is the first time you run it, it will ask you to create a new superuser. For development, you should do so using an easy username and password (you will have to enter it a lot!). (I typically use something like "foo".)

Notice that part of this process checks to see if there are any existing migrations and applies them.

That's it! We should have a perfectly synchronized database now.

## Running our Application

If all has gone well we can fire up our application and take it for a spin. Type:

`python manage.py runserver`

You should see this:

```
Performing system checks…

System check identified no issues (0 silenced).
April 13, 2015 - 17:10:18
Django version 1.8, using settings 'codekeeper.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

If you open Chrome and navigate to `http://localhost:8000/` you should be greeted with "Hello World" in big letters. Success!

## Writing Views

In our previous example we wrote a generic Django class-based view to handle a simple request for the home page, and displayed  "Hello World", properly formatted in HTML. However, in keeping with the idea of creating a Browseable API, we should think about making our home page useful for both humans *and* computers.

One of the best ways to start with this is to ensure our API is *self-describing*; that is, a machine can visit our site and configure its behaviour dynamically, according to the types of data it can retrieve. The easiest way to do this is to ensure that we provide hints on our machine-readable version of the places it can look for more information.

At the same time, we are still building a human-readable website, so we need to ensure our system is useable for people too. In many other packages, the API and human versions of the site needed completely different systems, but with Django REST framework its easy to build one view that adapts its behaviour to deliver either human or machine-readable data, as needed.

Let's first start with a look at the snippet view.

`views/snippet.py`

    from rest_framework import generics
    from rest_framework import renderers
    from codekeeper.models.snippet import Snippet
    from codekeeper.serializers.snippet import SnippetSerializer
    from codekeeper.renderers.custom_html_renderer import CustomHTMLRenderer


    class SnippetList(generics.ListCreateAPIView):
        template_name = "snippet/snippet_list.html"
        renderer_classes = (CustomHTMLRenderer,
                            renderers.JSONRenderer,
                            renderers.BrowsableAPIRenderer)
        model = Snippet
        serializer_class = SnippetSerializer
        queryset = Snippet.objects.all()


    class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
        template_name = "snippet/snippet_detail.html"
        renderer_classes = (CustomHTMLRenderer,
                            renderers.JSONRenderer,
                            renderers.BrowsableAPIRenderer)
        model = Snippet
        serializer_class = SnippetSerializer
        queryset = Snippet.objects.all()

`SnippetList` and `SnippetDetail` each deal with delivering the requested data back to the user; the list view returns a list of all snippets, while the detail view returns a single snippet.

The `renderer_classes` on each of these views determines the types of data each view will return. These particular views have three possible returns: `CustomHTMLRenderer` (more on this in a moment), `JSONRenderer` and `BrowsableAPIRenderer`. This means that each view is available in HTML, JSON, and REST Framework's built-in browseable API view.

The `CustomHTMLRenderer` is an extension of the REST Framework's `TemplateHTMLRenderer`. Create a file in your `renderers` directory called `custom_html_renderer.py` and add to it the following code:

`renderers/custom_html_renderer.py`

    from rest_framework.renderers import TemplateHTMLRenderer


    class CustomHTMLRenderer(TemplateHTMLRenderer):
        def render(self, data, accepted_media_type=None, renderer_context=None):
            """
            Renders data to HTML, using Django's standard template rendering.
            The template name is determined by (in order of preference):
            1. An explicit .template_name set on the response.
            2. An explicit .template_name set on this class.
            3. The return result of calling view.get_template_names().
            """
            renderer_context = renderer_context or {}
            view = renderer_context['view']
            request = renderer_context['request']
            response = renderer_context['response']

            if response.exception:
                template = self.get_exception_template(response)
            else:
                template_names = self.get_template_names(response, view)
                template = self.resolve_template(template_names)

            context = self.resolve_context({'content': data}, request, response)
            return template.render(context)

The purpose of this custom renderer is simple, and can be found on the second-last line of the `render` method:

    context = self.resolve_context({'content': data}, request, response)

This exposes the `content` variable in the templates containing the response data. (Without this it is difficult to get to the particular data in the response in the template).

For the templates, you should create a folder in your `templates` folder called `snippets` and in this folder create two files: `snippet_list.html` and `snippet_detail.html`. These will contain the code for templating the list and detail views, respectively.

For now, you can do a very simple placeholder template.

`templates/snippet/snippet_list.html`

    {% extends "base.html" %}

    {% block body %}
    <h1>List</h1>
    <ul>
    {% for snippet in content %}
        <li><a href="{{ snippet.url }}">{{ snippet.title }}</a></li>
    {% endfor %}
    </ul>

    {% endblock %}

`snippet_detail.html`

    {% extends "base.html" %}

    {% block body %}
        <h1>Detail</h1>
        <h2>Snippet title: {{ content.title }}</h2>
        <p>{{ content.snippet }}</p>
        <p>{{ content.creator }}</p>

    {% endblock %}

> Notice that in the List template, we are iterating through a list of snippets in "content", while in the detail template,
> "content" is the snippet object itself.

Next, let's wire this up in our `urls.py`. 

`urls.py`

    from django.conf.urls import patterns, include, url
    from django.contrib import admin

    from codekeeper.views.home import HomePageView
    from codekeeper.views.snippet import SnippetList, SnippetDetail

    urlpatterns = patterns('',
        url(r'^$', HomePageView.as_view(), name="home"),
        url(r'^snippets/$', SnippetList.as_view(), name="snippet-list"),
        url(r'^snippet/(?P<pk>[0-9]+)/$', SnippetDetail.as_view(), name="snippet-detail"),
        url(r'^admin/', include(admin.site.urls)),
    )

Before we continue, we'll need to also create the same view for our `Person` objects. This is because in order to render a `Snippet`, Django REST Framework needs to *also* need to know how to render a `Person` since they are related through a ForeignKey relationship.

Our `views/person.py` file looks very similar to our `snippet.py`

`views/person.py`:

    from rest_framework import generics
    from rest_framework import renderers
    from codekeeper.models.person import Person
    from codekeeper.serializers.person import PersonSerializer
    from codekeeper.renderers.custom_html_renderer import CustomHTMLRenderer


    class PersonList(generics.ListCreateAPIView):
        template_name = "person/person_list.html"
        renderer_classes = (CustomHTMLRenderer,
                            renderers.JSONRenderer,
                            renderers.BrowsableAPIRenderer)
        model = Person
        serializer_class = PersonSerializer
        queryset = Person.objects.all()


    class PersonDetail(generics.RetrieveUpdateDestroyAPIView):
        template_name = "person/person_detail.html"
        renderer_classes = (CustomHTMLRenderer,
                            renderers.JSONRenderer,
                            renderers.BrowsableAPIRenderer)
        model = Person
        serializer_class = PersonSerializer
        queryset = Person.objects.all()

Add these lines to your `urls.py` (making sure you import the views!):

    url(r'^people/$', PersonList.as_view(), name="person-list"),
    url(r'^person/(?P<pk>[0-9]+)/$', PersonDetail.as_view(), name="person-detail"),

*Note*: Notice the differences in plurals (for lists) and singulars, especially for "people" vs. "person".

Do the same for `views/tag.py`. By this point you should know what to do, so I won't reproduce the code here.

Next, modify your `view/home.py` to add the new views we've created to the Response object in the `get` method:

        def get(self, request, *args, **kwargs):
            response = Response({
                'snippets': reverse('snippet-list', request=request),
                'tags': reverse('tag-list', request=request),
                'people': reverse('person-list', request=request)
            })
        return response

Once all of this is in place, make sure your development server is still running and then refresh the page. You should still see your "Hello World" message. However, the real magic happens when you append `?format=json` to your URL:

![Figure 5](figures/figure5.png)

The same view, with a different format flag, gives us a JSON-formatted response! To really play with this, try changing the format flag to `api`:

![Figure 5b](figures/figure5b.png)

This is the Django REST Framework's "Browsable API" template, and it can be a pretty useful tool for poking around a REST API.

Note: Make sure to use Google Chrome because the links may not work in Safari.
This is the beginning of our API! Clicking on any of the links will bring you to a blank page, but that's because we have no content in our system yet.

# The Django Admin interface

Before we continue, let's look at the Django Admin interface. This will allow us to directly enter data into our database with a tool that comes with Django.

Finally, create a new folder, 'codekeeper/admin' and add two new files, `__init__.py` and `admin.py`.

Create the following in `admin/admin.py`:

        from django.contrib import admin

        from codekeeper.models.snippet import Snippet
        from codekeeper.models.person import Person
        from codekeeper.models.tag import Tag

        @admin.register(Snippet)
        class SnippetAdmin(admin.ModelAdmin):
            pass

        @admin.register(Person)
        class PersonAdmin(admin.ModelAdmin):
                pass

        @admin.register(Tag)
        class TagAdmin(admin.ModelAdmin):
                pass


In the `__init__.py` file add the following line:

        from codekeeper.admin import admin

Start your development server and point your web browser to `http://localhost:8000/admin`. You should be brought to a log-in page, where you should enter the username and password you entered when you synchronized your database.

Once in you should be at a screen that looks like this:

![Figure 6](figures/figure6.png)

Clicking on any of the content types will bring you to a screen where we can add or edit records.

Before continuing, notice that "Persons" is not properly named -- it should be "People". Django did its best to guess the plural form, but sometimes it gets it wrong. Let's fix this up.

Go to your `models/person.py` file and change your `Meta` class to the following:

        class Meta:
            app_label = "codekeeper"
            verbose_name_plural = "people"

If you did not quit your development server in your terminal, you should now just be able to refresh the page and see your changes.

![Figure 6b](figures/figure6b.png)

Now, let's create some dummy data to play with.

## Entering data

Start with the Snippet entry and create a couple code snippets. It doesn't matter what you enter; this is just to get a feel for how this site will work.

You will notice that you need to specify a creator for this snippet, and that there is an optional place to supply tags for the snippet. The "+" sign next to each of these fields allows you to add new entries to these tables in place.

After you've got all of your data entered you can manually check the data representations in your front-end. Go to `http://localhost:8000/snippets/?format=json` and choose to see what data is reflected in your API. It might look something like this:

![Figure 7](figures/figure7.png)

It's not pretty, but it's a start.

## Poking Around

Let's pause for a moment and review where we are now.

We have an application with some basic testing data in it that allows us to view and browse the site in a raw data form. We have the Django Admin interface up and running, and we can display the data in HTML, JSON, and the Browsable API.

Next, let's have a look at a useful little command-line tool that we can use to dig into the API a bit deeper.

## cURL

In the previous part of this tutorial the command-line URL utility `curl` was introduced as a way of interacting with an HTTP Server. In this section we'll look at some of the more advanced uses of cURL for poking around an API. 

Let's start with the basics:

        $> curl -XGET http://localhost:8000/

This should retrieve the HTML version of your home page. Pretty straightforward. Of course, we can also do this:

        $> curl -XGET http://localhost:8000/\?format\=json

to retrieve the JSON response. However, this isn't really RESTful. We're controlling the response type by explicitly defining it in the URL, which is mixing the HTTP layer with the request layer. This is a useful shortcut for seeing our JSON output in the browser, since we can't easily control the response type in a browser, but with cURL we can ask for a specific format to be returned without embedding it in the URL itself. To do this, we let the server know that we specifically want the JSON representation by passing in the `Accept:` header type:

        $> curl -XGET -H "Accept: application/json" http://localhost:8000/

Which should return the same response. Let's look at the verbose output to see a bit more of what's happening here:

    $> curl -v -XGET -H "Accept: application/json" http://localhost:8000/

    * Hostname was NOT found in DNS cache
    * Trying 127.0.0.1…
    * Connected to localhost (127.0.0.1) port 8000 (#0)
    > GET / HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: localhost:8000
    > Accept: application/json
    >
        * HTTP 1.0, assume close after body
    < HTTP/1.0 200 OK
    < Date: Mon, 13 Apr 2015 19:20:54 GMT
    < Server: WSGIServer/0.2 CPython/3.4.2
    < Vary: Accept, Cookie
    < Content-Type: application/json
    < Allow: GET, HEAD, OPTIONS
    < X-Frame-Options: SAMEORIGIN
    <
    * Closing connection 0
    {"tags":"http://localhost:8000/tags/","snippets":"http://localhost:8000/snippets/","people":"http://localhost:8000/people/"}

A few things should make a bit more sense to you now:

1. The `-H "Accept: application/json"` argument added the `Accept: application/json` request header to the outgoing request.
2. The server responded with a `200 OK` status code to let us know that everything's cool, yo.
3. The server has also passed on some "By the way…" information by letting us know the HTTP verbs that this endpoint accepts ("GET, HEAD, OPTIONS").
4. Finally, the actual content of the message is sent back in the message body at the very end.

Next, let's have a look at one of our existing snippets:

    $> curl -XGET -H "Accept: application/json" http://localhost:8000/snippet/1/

This should give us the data that we asked for. However, what if we wanted to delete this snippet? How do we do this programmatically?

    $> curl -v -XDELETE -H "Accept: application/json" http://localhost:8000/snippet/1/

    *   Trying 127.0.0.1…
    * Connected to localhost (127.0.0.1) port 8000 (#0)
    > DELETE /snippet/1/ HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: localhost:8000
    > Accept: application/json
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 204 NO CONTENT
    < Date: Mon, 13 Apr 2015 19:28:13 GMT
    < Server: WSGIServer/0.2 CPython/3.4.2
    < Vary: Accept, Cookie
    < Content-Length: 0
    < Allow: GET, PUT, PATCH, DELETE, HEAD, OPTIONS
    < X-Frame-Options: SAMEORIGIN
    <
    * Closing connection 0

What happened? The answer is somewhat hidden in the status code: 

    < HTTP/1.0 204 NO CONTENT

Based on this response we know two things: 1) the request was successful, since it's a 2XX status code, and 2) the response had no content. Let's see what happens when we try to fetch this snippet again:

    $> curl -XGET -H "Accept: application/json" http://localhost:8000/snippet/1/

In our response we see:

    < HTTP/1.0 404 NOT FOUND

Buh bye snippet 1!

Of course, this is problematic in any public-facing website, because we don't want just anyone sending DELETE requests to our data. We'll get to security later in the tutorial, though.

## Displaying data in the templates

Let's take a closer look at the `renderers/custom_html_renderer.py` file and the class we defined in it. You'll notice that towards the end it looks like this:

    context = self.resolve_context({'content': data}, request, response)
    return template.render(context)

The `context` variable is what is responsible for passing along the data from our view to the template. The most important thing to note here is the `content` key word. This is the variable that will allow us to access all the data that has been passed through this renderer and into our template system.

> Note: If you're ever wondering what fields, exactly, are available in your
> template, you can print the `content` variable directly by rendering it
> in a Django template. Just put `{{ content }}` in your template and refresh.

To see how this might work, open up the `snippet/snippet_list.html` template file and replace the content of the "body" block with the following Django template code:

    {% block body %}
        <h1>List</h1>
        <ul>
        {% for snippet in content %}
            <li>{{ snippet.title }}</li>
        {% endfor %}
        </ul>
    {% endblock %}

This will render a list of the snippets we have in our database in our web browser. It doesn't look like much, but we know it works.

![Figure 8](figures/figure8.png)

Let's bring in some CSS and JavaScript libraries to start making this look a little better.

## Bootstrap and jQuery

Twitter Bootstrap is a collection of CSS styles and JavaScript scripts that make creating a good-looking website relatively easy. It has styles for buttons and other form controls, as well as a powerful grid system for creating a pleasing layout.

You should begin by downloading the files from the [Bootstrap Website](http://getbootstrap.com).

You will have three folders, `css`, `js`, and `fonts`. You should move the contents of `css` and `js` to your existing folders in your `static` folder, and then copy the whole `fonts` directory to your `static` folder.

[jQuery](http://jquery.com) is a JavaScript library that makes dealing with JavaScript a lot easier. You should download both the 'compressed' and 'uncompressed' versions and put them in your `static/js` folder as `jquery.js` and `jquery.min.js`. jQuery will also complain if it can't find it's "map" file, so you should download the map file as well and put it in `static/js`.

To hook these up we just edit our `base.html` file and import the files.

Edit your `base.html` file to include these files like this:

```
{% load staticfiles %}
<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="utf-8">

    <title>Code Keeper Snippets</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.css' %}">
    <link rel="stylesheet" href="{% static 'css/bootstrap-theme.css' %}">
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
</head>

<body>
<div class="container">
    <div class="page-header">
        <div class="row">
            <div class="col-lg-2">
                <img src="{% static 'img/codekeeper.png' %}" />
            </div>
            <div class="col-lg-10">
                <h1>CodeKeeper</h1>
                <p class="lead">Keep your code</p>
            </div>
        </div>
    </div>
    {% block body %}

    {% endblock %}
</div>
<script src="{% static 'js/jquery.min.js' %}"></script>
<script src="{% static 'js/bootstrap.min.js' %}"></script>
<!--[if lt IE 9]>
<script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
<script src="{% static 'js/scripts.js' %}"></script>

</body>
</html>
```

> You may be wondering why we include the JavaScript at the end and not in the header?
> This is an optimization technique; since some JavaScript files can take a long
> time to load, we place them towards the end of the page so that the browser can load and
> display the page before starting to download the page. While it doesn't make it load
> faster, it gives the user the impression that it is loading faster since the
> page will show up sooner; otherwise, the user will have to wait until all the files in the
> header have loaded before they see the page contents.

Note that we've added a little logo in our `static/img` folder. You should do the same.

Just to round it out, let's update our `index.html` template to allow users to click links to browse our site:

```
{% extends "base.html" %}

{% block body %}
<h1>Explore our site</h1>
<ul>
    <li><a href="/snippets/">Snippets</a></li>
    <li><a href="/tags/">Tags</a></li>
    <li><a href="/people/">People</a></li>
</ul>
{% endblock %}
```

Visiting the website in a browser will now display our base template with the data from each of the other templates in the body region. Now it's starting to come together.

## Templating the detail and list pages

To save space and time writing this section, I won't go through every step of the customization and design of the pages, but I will demonstrate how to theme the "snippet" list and detail pages. We can expand the templates created previously and add some basic information display. Remember that we are accessing everything on the model through the `content` variable. 

Open up the `snippet/snippet_list.html` page. We will make a small change to link each item in the list to an item page.

```
{% extends "base.html" %}

{% block body %}
    <h1>Snippets</h1>
    <ul>
    {% for snippet in content %}
        <li><a href="{{ snippet.url }}">{{ snippet.title }}</a></li>
    {% endfor %}
    </ul>
{% endblock %}
```

If you refresh your snippet list page now, the items in the list should be hyperlinked. Clicking on the link will show you a blank page. To customize this look, add the following code to `snippet/snippet_detail.html`.

```
{% extends "base.html" %}

{% block body %}
    <h1>Snippet Detail</h1>
    <dl>
        <dt>Snippet title</dt>
        <dd>{{ content.title }}</dd>
        <dt>Snippet</dt>
        <dd>{{ content.snippet }}</dd>
        <dt>Creator</dt>
        <dd>{{ content.creator }}</dd>
    </dl>
{% endblock %}
```

Notice here that we use the `content` variable to access the fields of the model that we are displaying.

If you refresh the detail page for one of your snippets, you should now see a display of the information contained in your database for each entry in your snippet table.

You now have the basics for inserting data into a template, so you can build the list and detail views on your own. If you get stuck you should refer to the templates in the GitHub repository for this tutorial for any further changes and modifications.

# Extended Serializers

It's time we took a closer look at serializers.

Let's imagine that I would like to display the tags for each snippet. Currently, the only data the serializer is giving me is a URL to retrieve the tag record -- not the actual tag name itself.

Here is a cURL request and a JSON response of a snippet to illustrate:

```
$> curl -XGET -H "Accept: application/json" http://localhost:8000/snippet/2/

{
    "url":"http://localhost:8000/snippet/2/",
    "title":"Another snippet",
    "snippet":"def foo():\r\n    print(\"Python is Cool!)\r\n",
    "created":"2015-04-13T19:08:37.029027Z",
    "updated":"2015-04-13T19:08:37.029073Z",
    "creator":"http://localhost:8000/person/1/",
    "tags":["http://localhost:8000/tag/1/"]
}
```

To get the tag name, we can embed a tag serializer within our snippet serializer. Open up `serializers/snippet.py` and create a new serializer for your tag data. Your file should look like this:

    from codekeeper.models.snippet import Snippet
    from codekeeper.models.tag import Tag
    from rest_framework import serializers

    class TagSnippetSerializer(serializers.HyperlinkedModelSerializer):
        class Meta:
            model = Tag

    class SnippetSerializer(serializers.HyperlinkedModelSerializer):
        tags = TagSnippetSerializer(many=True)

        class Meta:
            model = Snippet

Now the request for the same activity results in this:

    {
        "url":"http://localhost:8000/snippet/2/",
        "tags":[
            {
                "url":"http://localhost:8000/tag/1/",
                "name":"Code",
                "created":"2015-04-13T19:06:08.498891Z",
                "updated":"2015-04-13T19:06:08.498935Z"
            },
            {
                "url":"http://localhost:8000/tag/2/",
                "name":"Cool Stuff",
                "created":"2015-04-13T19:06:14.582711Z",
                "updated":"2015-04-13T19:06:14.582753Z"}
        ],
        "title":"Another snippet",
        "snippet":"def foo():\r\n    print(\"Python is Cool!)\r\n",
        "created":"2015-04-13T19:08:37.029027Z",
        "updated":"2015-04-13T19:08:37.029073Z",
        "creator":"http://localhost:8000/person/1/"
    }

We can now access the title of our tag through the `tags` field. Open your `templates/snippet/snippet_list.html` and change the list to a table, adding a column for the tags.

    {% extends "base.html" %}

    {% block body %}
    <h1>Snippets</h1>
    <table class="table">
        <thead>
            <tr>
                <th>Snippet</th>
                <th>Tags</th>
            </tr>
        </thead>
        <tbody>
            {% for snippet in content %}
                <tr>
                    <td><a href="{{ snippet.url }}">{{ snippet.title }}</a></td>
                    <td>
                        <ul>
                            {% for tag in snippet.tags %}
                            <li><a href="{{ tag.url }}">{{ tag.name }}</a></li>
                            {% endfor %}
                        </ul>
                    </td>
                </tr>
            {% endfor %}
        </tbody>
    </table>

    {% endblock %}

Notice that our tags are now displayed as a list in the table, and that clicking on them brings you to the page for that tag.

## Computed Fields

In our models we can define methods that can be used to process and extract information stored in that model. We have already seen a very simple example of this with the `__str__` method on our models:

    class Person(models.Model):
        ...
        def __str__(self):
            return "{0}, {1}".format(self.last_name, self.first_name)

In other words, a computed field is a model method that acts like a field. To illustrate, let's make computed field that can display the person's name as "Firstname Lastname":

    @property
    def full_name(self):
        return "{0} {1}".format(self.first_name, self.last_name)

We can now use this in our serializers to deliver the person's full name in "natural" order:

Add the following new serializer to your `serializers/snippet.py`:

    class CreatorSnippetSerializer(serializers.HyperlinkedModelSerializer):
        full_name = serializers.ReadOnlyField()
        class Meta:
            model = Person

And then change your SnippetSerializer to include this serializer for the `creator` field:

    class SnippetSerializer(serializers.HyperlinkedModelSerializer):
        tags = TagSnippetSerializer(many=True)
        creator = CreatorSnippetSerializer()

        class Meta:
            model = Snippet

Now we have a new field in our output, `full_name` which we can use to display a more natural, human-friendly version of the creator's name.

    $> curl -XGET -H "Accept: application/json" http://localhost:8000/snippet/2/

    {
        url: "http://localhost:8000/snippet/2/",
        tags: [
        {
                url: "http://localhost:8000/tag/1/",
                name: "Code",
                created: "2015-04-13T19:06:08.498891Z",
                updated: "2015-04-13T19:06:08.498935Z"
            },
            {
                url: "http://localhost:8000/tag/2/",
                name: "Cool Stuff",
                created: "2015-04-13T19:06:14.582711Z",
                updated: "2015-04-13T19:06:14.582753Z"
            }
        ],
        creator: {
            url: "http://localhost:8000/person/1/",
            full_name: "Andrew Hankinson",
            first_name: "Andrew",
            last_name: "Hankinson",
            created: "2015-04-13T17:18:39.460934Z",
            updated: "2015-04-13T17:18:39.460979Z"
        },
        title: "Another snippet",
        snippet: "def foo(): print("Python is Cool!) ",
        created: "2015-04-13T19:08:37.029027Z",
        updated: "2015-04-27T19:25:26.486076Z"
    }

There's a lot of useless information in there, though, so let's remove the 'created' and 'updated' fields from the output by changing our `CreatorSnippetSerializer` to specify the fields that we want:

    class CreatorSnippetSerializer(serializers.HyperlinkedModelSerializer):
        full_name = serializers.ReadOnlyField()
        class Meta:
            model = Person
            fields = ('url', 'full_name')


And finally, let's add this to our snippet list table by adding another column to the table:

![Figure 8b](figures/figure8b.png)

It's starting to look like a real website!

# Migrations

We briefly touched on migrations previously when we were setting up our database, but didn't get a chance to dive into them to see what they can do.

To re-cap, migrations allow you to update your models and synchronize your changes to your database without having to dump and re-import your data. They're a huge timesaver and well worth understanding.

Let's suppose that we realize that we would really like to describe the programming language of a particular snippet. Right now we *could* use the tags, but that might get a bit messy. So, let's add a new model, "Language" and add a foreign key to our existing Snippet model.

`models/language.py`:

    from django.db import models


    class Language(models.Model):
        class Meta:
            app_label = "codekeeper"

        name = models.CharField(max_length=128)
        created = models.DateTimeField(auto_now_add=True)
        updated = models.DateTimeField(auto_now=True)

        def __str__(self):
            return "{0}".format(self.name)

(Don't forget to update your `models/__init__.py`!)

`models/snippet.py`

    ...
    language = models.ForeignKey("codekeeper.Language")
    ...

After making these changes, refresh your application in your browser. You should get an error that says something along the lines of `django.db.utils.OperationalError: no such column: codekeeper_snippet.language_id`

You've added the field to our model, but now we need to synchronize our model changes to our underlying database. To do that we make a migration by using the manage command.

    $> python manage.py makemigrations
    ... stuff ...

If you look in your `codekeeper/migrations` folder you will see a new file. This Python code was auto-generated and will help migrate your data from the old structure to the new database structure. To run the migration:

    $> python manage.py migrate
    ... stuff ...

Refreshing your snippet view now should not give you the database error, but it will give you an error about not being able to find the view 'language-detail'. This is related to your application not being able to find out how to serialize the data in your new language model. This is a path we've been down before, so getting your application up and running is now an exercise for the reader.

Once you have your site back up and running we'll leave our Django web application alone and start looking at the search component of our website using a new piece of software, Solr.
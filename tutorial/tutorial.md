We start with creating a project, and a blog app, and installing the libraries we need.
- `pip install django django-rest-framework django-cors-headers djoser`
- `django-admin startproject pagination_example`
- `python manage.py startapp blog`

- **Creating a simple `Post` model inside `blog/models.py`**
    ```
    from django.db import models

    class Post(models.Model):
        title = models.CharField(max_length=255)
    ```
- Add apps to `pagination_example/settings.py`
    ```
    # Application definition

    INSTALLED_APPS = [
        ...
        'django.contrib.staticfiles',
        'rest_framework',
        'blog.apps.BlogConfig',
    ]

    ```
- **Update the db**
    - `python manage.py makemigrations`
    - `python manage.py migrate`
- **Run the server in the background**
    - `python manage.py runserver` 

- create `templates/blog` folder under `blog` folder. (`blog/templates/blog`)
- create `index.html` (Created a page with vue inside it and Blog object.)
    ```
    <!DOCTYPE html>
    <html>

    <body>
        <h1>Hello World!</h1>

        <div id="blog">


        </div>
        <script src="https://unpkg.com/vue@next"></script>
        <script>
            const Blog = {
                data() {
                    return {

                    }
                },
                mounted() {
                    console.log('Hello!')
                },
            }

            Vue.createApp(Blog).mount('#blog')
        </script>
    </body>

    </html>
    ```
- open `blog/views.py`
    ```
    from django.shortcuts import render


    def index(request):
        return render(request, 'blog/index.html')

    ```
- create `blog/urls.py`
    ```
    from django.urls import path, include

    from . import views

    urlpatterns = [
        path('', views.index, name='index')
    ]
    ```
- open `pagination_example/urls.py`
- add the new paths
    ```
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include("blog.urls"))
    ]

    ```
- open `localhost:8000` to check if you see the index page.
- create a `serializer` for the posts.
    - create `blog/serializers.py` 
    ```
    from django.db.models.base import Model
    from rest_framework import serializers

    from .models import Post


    class PostSerializer(serializers.ModelSerializer):
        class Meta:
            model = Post
            fields = ('id', 'title')

    ```
- open `views.py` again.
    - add some imports, and create two classes.

    ```
    from django.shortcuts import render

    from rest_framework import viewsets
    from rest_framework.pagination import PageNumberPagination

    from .models import Post
    from .serializers import PostSerializer

    def index(request):
        return render(request, 'blog/index.html')

    class PostPagination(PageNumberPagination):
        page_size = 5

    class PostViewSet(viewsets.ModelViewSet):
        pagination_class = PostPagination
        serializer_class = PostSerializer
        queryset = Post.objects.all()

    ```

    - *elaboration*: There is a lot of magic happening in the background.
        - `PageNumberPagination` will make sure that we will get only 5 items per page.
- open `blog/urls.py` again

    -  add router and urls of the router.
    ```
    from pagination_example.blog.models import Post
    from django.urls import path, include
    from rest_framework.routers import DefaultRouter

    from . import views

    router = DefaultRouter()
    router.register('posts', views.PostViewSet, basename='posts')

    urlpatterns = [
        path('', views.index, name='index'),
        path('', include(router.urls))
    ]

    ```
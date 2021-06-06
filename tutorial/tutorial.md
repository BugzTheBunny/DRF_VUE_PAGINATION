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

- Adding some posts:
    - go to `http://localhost:8000/posts/`
    - add some posts by writing a title, and pressing the "post" button. **Add about 15 posts**
    - go to `http://localhost:8000/posts/` again, and you will see that you get links to the next page, and each request has 5 items.
    you will see something like this:
    ```
    HTTP 200 OK
    Allow: GET, POST, HEAD, OPTIONS
    Content-Type: application/json
    Vary: Accept

    {
        "count": 11,
        "next": "http://localhost:8000/posts/?page=2",
        "previous": null,
        "results": [
            {
                "id": 1,
                "title": "P1"
            },
            {
                "id": 2,
                "title": "P12"
            },
            {
                "id": 3,
                "title": "P123"
            },
            {
                "id": 4,
                "title": "P124"
            },
            {
                "id": 5,
                "title": "P10"
            }
        ]
    }
    ```
    If you see this, then go to the next step.
- open `index.html`
- now we gonna add the data retrival and implement the pagination.
    - edit the script like as a start like this:
    ```
    <script>
        const Blog = {
            data() {
                return {
                    posts: [],
                    currentPage:1
                }
            },
            mounted() {
                this.getPosts()
            },
            delimiters: ['[[', ']]'],
            methods: {
                getPosts(){
                    fetch('/posts/').then(
                        response => {
                            return response.json()
                        }
                    ).then(data => {
                        console.log(data)
                        this.posts = data.results
                    }).catch(error => {
                        console.log(error)
                    })
                }
            }
        }

        Vue.createApp(Blog).mount('#blog')
    </script>
    ```
    - *elaboration*: we added at this point the data retrival from the backend, we use `fetch` function which is a built-in function if all of the browsers to fetch data, we use this instead of axios, in another tutorial i will show it with axios and Vue3 CLI.
        - `posts: []` : the array which will contain our list of posts
        - `mounted` : tells what will happen when the page loads.
        - `delimiters` : defines for Django where to show the data.
        - `getPosts` : the function iself that gets the data from the backend.
        - `this.posts = data.results` : here we set the retrived posts, to the `posts` list, after which they are displayed in the page.
    - if you refresh the page `localhost:8000` and open the console, you will see taht we are getting the data object, will look something like this: (the content will not be exactly as mine, it will contain the data you added)
        ```
        {count: 11, next: "http://localhost:8000/posts/?page=2", previous: null, results: Array(5)}
        count: 11
        next: "http://localhost:8000/posts/?page=2"
        previous: null
        results: Array(5)
        0: {id: 1, title: "P1"}
        1: {id: 2, title: "P12"}
        2: {id: 3, title: "P123"}
        3: {id: 4, title: "P124"}
        4: {id: 5, title: "P10"}
        length: 5
        __proto__: Array(0)
        __proto__: Object
        ```
- **Now let's see the data outside of the console!**
    - go to the `<div id="blog">` block, and add some things.
    ```
    <h1>Hello World!</h1>
    <div id="blog">
        <div v-for="post in posts" v-bind:key="post.id">
            <h2>[[ post.title]] </h2>
        </div>
    </div>
    ```
    - at this point, you should see 5 items in the page.
- **Adding the exploration of the next page**
    - add a template below the div we created above.
    ```
        <div v-for="post in posts" v-bind:key="post.id">
            <h2>[[ post.title]] </h2>
        </div>
        <template v-if="showNextButton">
            <button @click="loadNext()">Next</button>
        </template>
    ```
    -**elaboration**: we created a template, which will show the next button if we actually have a link to the next button.
    the v-if will check if we have next page link.
    on the button, we have a @click="loadNext()" which will load the next data for us when we click it. **We still did not create `showNextButton` or `loadNext()` at this point, so don't worry.
- Go back to the script again, and add another data field (`showNextButton`)
    we set it to false, because we want it as false by default, unless there is a link.
    ```
    ...
    return {
        posts: [],
        currentPage: 1,
        showNextButton: false <- this thing
    ...
    ```
- Edit the `getPosts()` function
    we are adding a check, if we have next in the data that we are getting as a response.
    first we set the showNextButton to false, and if we have some data, it will be true.
    ```
    ...
    ).then(data => {
    
        this.showNextButton = false             <- Add this

        if (data.next) {                        <- Add this
            this.showNextButton = True          <- Add this
        }                                       <- Add this

        this.posts = data.results
    ...
    ```
- Add another function above the `getPosts()`
     ```
    loadNext() {
        this.currentPage += 1
        this.getPosts()
    },
    ```
- Edit the link that we use fetch on it, so we will be able to use the `currentPage` variable over there.
    ```
    fetch(`/posts/?page=${this.currentPage}`).then(
    ```
    - notice that we changed the ' for `, which allows us to format the string.
    - now everytime we use "next" button, our `currentPage` is incremented by 1, and after that happens, we fetch the data from the url with this page.
- At this point, everything works awesome, and you can navigate forward.
    - I would recommand you to try to create the previous button and functions now by yourself, if not, just follow from here quickly.

    - add a template.
    ```
    <template v-if="showPrevButton">
        <button @click="loadPrev()">Previous</button>
    </template>
    ```
    - add the variable
    ```
    currentPage: 1,
    showNextButton: false,
    showPrevButton: false                   <- Add this
    ```
    - add the reseting
    ```
    this.showNextButton = false
    this.showPrevButton = false             <- Add this


    if (data.next) {
        this.showNextButton = true
    }

    if (data.previous) {                    <- Add this
        this.showPrevButton = true          <- Add this
    }

    this.posts = data.results
    ```
    - add the function to decrease the page number
    ```
    loadNext() {
    this.currentPage += 1
    this.getPosts()
    },
    loadPrev() {                            <- Add this
        this.currentPage -= 1               <- Add this
        this.getPosts()                     <- Add this
    },                                      <- Add this
    ```

- Great! we are done! everything should be working
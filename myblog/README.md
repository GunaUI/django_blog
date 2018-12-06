# django_blog
* Create Project
* Create App 
* Inform project about the newly created app (refer settings.py INSTALLED_APPS)
* Do migrations steps
* Run server
## Models
* Create models in models.py
    * POST Model
    ```
        class Post(models.Model):
            author = models.ForeignKey('auth.User', on_delete=models.CASCADE)
            title = models.CharField(max_length=200)
            text = models.TextField()
            created_date = models.DateTimeField(default=timezone.now)
            published_date = models.DateTimeField(blank=True, null=True)

            def publish(self):
                self.published_date = timezone.now()
                self.save()

            def approve_comments(self):
                return self.comments.filter(approved_comment=True)

            def get_absolute_url(self):
                return reverse("post_detail",kwargs={'pk':self.pk})
            
            def __str__(self):
                return self.title
    ```
     * Comment Model

    ```
        class Comment(models.Model):
            post = models.ForeignKey('blog.Post', related_name='comments',on_delete=models.CASCADE,)
            author = models.CharField(max_length=200)
            text = models.TextField()
            created_date = models.DateTimeField(default=timezone.now)
            approved_comment = models.BooleanField(default=False)

            def approve(self):
                self.approved_comment = True
                self.save()

            def get_absolute_url(self):
                return reverse("post_list")
                
            def __str__(self):
                return self.text
    ```
    * get_absolute_url - inform model where to go back once done

## Form
* Now its time to create forms for the model
* Create new forms.py inside app
* here we going to use dictionary widgets to add special class to the form
    ```
        from django import forms

        from blog_app.models import Post, Comment

        class PostForm(forms.ModelForm):

            class Meta:
                model = Post
                fields = ('author','title', 'text',)

                widgets = {
                    'title': forms.TextInput(attrs={'class': 'textinputclass'}),
                    'text': forms.Textarea(attrs={'class': 'editable medium-editor-textarea postcontent'}),
                }

        class CommentForm(forms.ModelForm):

            class Meta:
                model = Comment
                fields = ('author', 'text',)

                widgets = {
                    'author': forms.TextInput(attrs={'class': 'textinputclass'}),
                    'text': forms.Textarea(attrs={'class': 'editable medium-editor-textarea'}),
                }

    ```
##  CSS
* Now create a new 'static' folder inside app to acomodate css ,js and other static files
* inside css folder make a new css file and apply all your css here
* create STATIC_ROOT in settings.py
    ```
        STATIC_ROOT = os.path.join(BASE_DIR,"static")
    ```
## Template
* Now create a new 'template' folder inside app
* Then create template directory path in settings.py
    ```
    TEMPLATE_DIR = os.path.join(BASE_DIR,'blog_app/templates/blog')
    ```
* Then register this TEMPLATE_DIR to templates
    ```
    'DIRS': [TEMPLATE_DIR,],
    ```
* Then register login redirect URL
    ```
        LOGIN_REDIRECT_URL = '/'
    ```
## Views and Templates
### About
* Create a simple  AboutView in views.py
    ```
    class AboutView(TemplateView):
        template_name = 'about.html'
    ```
* Create base.html and about.html inside app's template folder
* Create base template
* Extend base template in about.html
    ```
        {% extends "blog/base.html" %}
        {% block content %}
        <h1>About me</h1>
        <p>Thanks for visiting my blog! Check all this cool stuff about me!</p>

        {% endblock %}
    ```
* Register Url.py about this newly created views and templates
    ```
        path('', include('blog_app.urls')),
    ```
    ```
        path('about/', views.AboutView.as_view(),name='about'),
    ```
### Post List

* Create a PostListView in views.py
    ```
    class PostListView(ListView):
    model = Post

    def get_queryset(self):
        return Post.objects.filter(published_date__lte=timezone.now()).order_by('-published_date')
    ```
* In get_queryset lte stands for lesser than or equal tp and minus infornt of published_date bring latest post to top

* Add Post List in URL
    ```
        path('', views.PostListView.as_view(),name='post_list'),
    ```
### Post Detail

* Create a PostDetailView in views.py
    ```
    class PostDetailView(DetailView):
        model = Post
    ```
* Add Post Detail in URL
    ```
        path('post/<int:pk>',views.PostDetailView.as_view(),name='post_detail')
    ```
### CreatePostView

* Create a CreatePostView in views.py
    ```
    class CreatePostView(LoginRequiredMixin,CreateView):
        login_url = '/login/'
        redirect_field_name = 'blog_app/post_detail.html'

        form_class = PostForm

        model = Post
    ```
* here login required to proceed post in "class based view", for that we need to import LoginRequiredMixin for authentication purpose

    ```
    from django.contrib.auth.mixins import LoginRequiredMixin
    ```
* Incase if it is function based view with automated login funtionality add login_required

    ```
        from django.contrib.auth.decorators import login_required
    ```
* here PostForm refers to the form template
* Create Post Detail in URL
    ```
        path('post/new/',views.CreatePostView.as_view(),name='post_new')
    ```
### UpdatePostView

* Create a PostUpdateView in views.py
    ```
    class PostUpdateView(LoginRequiredMixin,CreateView):
        login_url = '/login/'
        redirect_field_name = 'blog_app/post_detail.html'

        form_class = PostForm

        model = Post
    ```
* Rest of the steps same as create

### DeletePostView
* Create delete post view , import reverse lazy to redirect after deleted succesfully untill we don't want to move to other page.

    ```
    class PostDeleteView(LoginRequiredMixin,DeleteView):
        model = Post
        success_url = reverse_lazy('post_list')
    ```
### DraftListView
* Create DraftListView with get_queryset published_date__isnull 

    ```
        class DraftListView(LoginRequiredMixin,ListView):
            login_url = '/login/'
            redirect_field_name = 'blog/post_list.html'

            model = Post

            def get_queryset(self):
                return Post.objects.filter(published_date__isnull=True).order_by('created_date')
    ```
* Add corresponding URL Path

    ```
        path('drafts/',views.DraftListView.as_view(),name='post_draft_list'),
    ```











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

## Comments

### Add comments to your post

* Create a add_comment_to_post function in views.py
* before that we need to import login_required, get_object_or_404, redirect

    ```
        @login_required
        def add_comment_to_post(request, pk):
            post = get_object_or_404(Post, pk=pk)
            if request.method == "POST":
                form = CommentForm(request.POST)
                if form.is_valid():
                    comment = form.save(commit=False)
                    comment.post = post
                    comment.save()
                    return redirect('post_detail', pk=post.pk)
            else:
                form = CommentForm()
            return render(request, 'blog_app/comment_form.html', {'form': form})
    ```
* here login_required decorator is need to comment on any post
* then to comment a post we need a request and post's primary key
* get_object_or_404 means get the Post obeject or return not found in that we have to pass POST model and Primart key
* Once comment form submited add primary key details ie "Post" and then confirm save untill commit comments to false
* Once sucessfully comments submited redirect to post_detail page
* incase if the post method is not "POST" just return it to the comment form with form context dictionary

* Add corresponding URL Path for comment post
    ```
        path('post/comment/<int:pk>/',views.add_comment_to_post,name='add_comment_to_post'),
    ```

### Approve comments

    ```
        @login_required
        def comment_approve(request, pk):
            comment = get_object_or_404(Comment, pk=pk)
            comment.approve()
            return redirect('post_detail', pk=comment.post.pk)
    ```
* Add corresponding URL Path for comment post
    ```
        path('comment/approve/<int:pk>/',views.comment_approve,name='comment_approve'),
    ```
### Remove comments
    ```
        @login_required
        def comment_remove(request, pk):
            comment = get_object_or_404(Comment, pk=pk)
            post_pk = comment.post.pk
            comment.delete()
            return redirect('post_detail', pk=post_pk)
    ```
* Add corresponding URL Path for comment post
    ```
        path('comment/remove/<int:pk>/',views.comment_remove,name='comment_remove'),
    ```
### Publish Post

    ```
        @login_required
        def post_publish(request, pk):
            post = get_object_or_404(Post, pk=pk)
            post.publish()
            return redirect('post_detail', pk=pk)
    ```
* Add corresponding URL Path for Publish Post 
    ```
        path('post/publish/<int:pk>/',views.post_publish,name='post_publish'),
    ```











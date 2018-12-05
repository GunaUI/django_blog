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








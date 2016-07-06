# djangorelishblog

### Install

1. Add **django.contrib.humanize**, **djangorelishblog** and **markdownx** to your `INSTALLED_APPS` setting:

    INSTALLED_APPS = [
        ...
        'django.contrib.humanize',
        'djangorelishblog',
        'markdownx',
    ]

2. Include the **djangorelishblog** and **mardownx** URLconf in your project urls.py:

    url(r'^blog/', include('djangorelishblog.urls')),
    url(r'^markdownx/', include('markdownx.urls')),

3. Run `python manage.py collectstatic` to setup **markdownx**.

4. Run `python manage.py migrate` to create the blog models.

5. Start the development server and visit http://127.0.0.1:8000/admin/
   to create a blog (you'll need the Admin app enabled).

6. Visit http://127.0.0.1:8000/blog/ to view the blog.


### Models

#### Post
##### Properties
* `author` - ForeignKey to `USER_AUTH_MODEL` in settings.
* `title` - CharField with a max length of 255.
* `slug` - SlugField with a max length of 255.
* `body` - MarkdownxField.
* `published` - BooleanField to say whether the post is viewable to everyone or not.
* `created_on` - DateField with `auto_now_add` set to `True`.

##### Virtual properties
* `intro` - returns a string representing a short extract of the body, up to the `[[ MORE_LINK ]]` tag.
* `full_body` - returns a string replacing the `[[ MORE_LINK ]]` tag.
* `url` - returns a string of the full URL of the post.

##### Manager
The `Post` model uses a custom objects manager that adds a couple of handy methods.
* `Post.objects.published()` - returns all `Post` objects that are viewable by all.
* `Post.objects.unpublished()` - returns all `Post` objects that are _not_ viewable by all.

###### Notes
* If you use a User model other than the standard Django User model, add `USER_AUTH_MODEL = <model name>` in your Django settings file.
* Place `[[ MORE_LINK ]]` tag part way through the body which will show as the intro to the post. This tag is removed in `full_body`.
* When presenting the post, use `full_body` instead of `body` as it will remove the `[[ MORE_LINK ]]` tag.
* When saving a `Post` object, ommit the `created_on` parameter as it will automatically be set by Django.

#### Comment
##### Properties
* `post` - ForeignKey to `Post` with a related name of 'comments'.
* `author_name` - CharField with a max length of 64.
* `author_email` - EmailField.
* `author_website` - URLField that can be blank and null.
* `body` - TextField.
* `created_on` - DateField with `auto_now_add` set to `True`.

### Forms

#### CommentForm
A simple form used at the bottom of a `Post` to allow a viewer to submit a comment.

Has the fields `author_name`, `author_email`, `author_website`, and `body`.

### Views

#### BlogView
`BlogView` is a subclass of django.views.generic.list.ListView` which diplays all `Post` objects that are `published`. It sets the `context_object_name` to `'posts'` so you can refer to `{{ posts }}` in your template, and uses the standard `blog.html` template shipped with `djangorelishblog`.

#### BlogPostView
`BlogPostView` is a subclass of `django.views.generic.base.View` which displays a single `Post` object, a `CommentForm` and all of the comments related to that `Post`.
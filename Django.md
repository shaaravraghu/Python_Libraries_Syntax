# Django: The Definitive Developer Reference

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Project Setup](#2-installation--project-setup)
- [3. URL Routing](#3-url-routing)
- [4. Views](#4-views)
- [5. Templates](#5-templates)
- [6. Models & ORM](#6-models--orm)
- [7. Migrations](#7-migrations)
- [8. Forms](#8-forms)
- [9. Admin](#9-admin)
- [10. Authentication & Authorization](#10-authentication--authorization)
- [11. Middleware](#11-middleware)
- [12. Static & Media Files](#12-static--media-files)
- [13. Sessions & Cookies](#13-sessions--cookies)
- [14. Caching](#14-caching)
- [15. Signals](#15-signals)
- [16. REST APIs with Django REST Framework (DRF)](#16-rest-apis-with-django-rest-framework-drf)
- [17. Testing](#17-testing)
- [18. Security](#18-security)
- [19. Deployment](#19-deployment)
- [20. Async Django](#20-async-django)
- [21. Performance Optimization](#21-performance-optimization)
- [22. Advanced Topics](#22-advanced-topics)

## 1. INTRODUCTION & HISTORY

### What is Django?
Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, allowing you to focus on writing your app without needing to reinvent the wheel. It's free and open source.

### MTV Architecture
Django follows the **Model-Template-View (MTV)** architecture, which is a slight variation of the well-known MVC (Model-View-Controller) pattern.
- **Model (M):** The data access layer. Handles database schema, queries, and business logic for data.
- **Template (T):** The presentation layer. Handles what the user sees (HTML, CSS, JS).
- **View (V):** The business logic layer. Acts as the controller, taking the user request, retrieving data from the Model, and passing it to the Template.

### Philosophy
- **DRY (Don't Repeat Yourself):** Every distinct concept and piece of data should live in one, and only one, place.
- **Batteries-Included:** Django comes with common web development tools out-of-the-box (ORM, Admin, Authentication, Forms, Sessions, etc.).
- **Explicit is better than implicit:** Django relies heavily on explicitly defined configuration rather than "magic" conventions.

### Version Timeline
- **2005:** Open-sourced (v0.90)
- **2008:** v1.0 Released
- **2017:** v2.0 Released (Python 3 only)
- **2019:** v3.0 Released (Async support begins)
- **2021:** v4.0 Released (Zoneinfo for timezones, Redis cache backend)
- **2024:** v5.0 Released (Form rendering improvements, DB computed defaults)

💡 **Pro Tip:** Always use LTS (Long-Term Support) versions (e.g., 3.2, 4.2) for enterprise applications to ensure extended security updates without breaking changes.

⚠️ **Common Pitfalls:** Trying to force Django into patterns from other frameworks (like Spring or Express). Embrace the "Django Way" (MTV, Admin, ORM) instead of fighting it.

---

## 2. INSTALLATION & PROJECT SETUP

### Installation & Virtual Environments
It is highly recommended to isolate Django projects using Python virtual environments.

```bash
# Create a virtual environment named 'venv'
python -m venv venv

# Activate the virtual environment (Windows)
venv\Scripts\activate
# Activate the virtual environment (Mac/Linux)
# source venv/bin/activate

# Install Django using pip
pip install django
```

### Project vs. App Structure
- **Project:** The entire web application, including configuration and multiple apps.
- **App:** A specific, self-contained module within the project (e.g., `blog`, `users`, `store`).

```bash
# Start a new Django project named 'core'
django-admin startproject core .

# Start a new app named 'blog'
python manage.py startapp blog
```

### Key `manage.py` Commands
```bash
python manage.py runserver      # Starts the development server
python manage.py makemigrations # Detects model changes and creates migration files
python manage.py migrate        # Applies migrations to the database
python manage.py createsuperuser # Creates an admin user
python manage.py shell          # Opens an interactive Python shell with Django context
python manage.py test           # Runs the test suite
```

### `settings.py` Deep Dive
The `settings.py` file controls everything in your project.

```python
# core/settings.py

import os
from pathlib import Path

# Base directory of the project
BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
# Used for cryptographic signing, sessions, CSRF, etc.
SECRET_KEY = 'django-insecure-your-secret-key'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

# Hosts that are allowed to serve this Django app
ALLOWED_HOSTS = ['127.0.0.1', 'localhost', '.yourdomain.com']

# Installed apps: core Django apps, third-party apps, and your local apps
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Local apps
    'blog',
]

# Middleware processes request/response globally
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# Root URL routing configuration
ROOT_URLCONF = 'core.urls'

# Database configuration (Defaults to SQLite for development)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

💡 **Pro Tip:** Use `django-environ` to keep secrets like `SECRET_KEY` and `DATABASES` out of source control. Use environment variables for different environments.

⚠️ **Common Pitfalls:** Leaving `DEBUG = True` in production. This exposes sensitive configuration and source code through traceback pages and creates memory leaks due to SQL query logging.

---

## 3. URL ROUTING

Django uses the `urls.py` file to map URL paths to view functions/classes.

### Root URL Configuration
```python
# core/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # include() delegates routing to the blog app's urls.py
    path('blog/', include('blog.urls')),
]
```

### App URL Configuration
```python
# blog/urls.py
from django.urls import path, re_path
from . import views

# App namespace for URL reversing
app_name = 'blog'

urlpatterns = [
    # path(route, view, kwargs=None, name=None)
    path('', views.post_list, name='post_list'),
    
    # URL Converters: <type:name>
    # str, int, slug, uuid, path
    path('post/<int:post_id>/', views.post_detail, name='post_detail'),
    path('post/<slug:slug>/', views.post_by_slug, name='post_by_slug'),
    path('user/<uuid:user_id>/', views.user_profile, name='user_profile'),

    # Regular Expressions with re_path (legacy/advanced usage)
    re_path(r'^archive/(?P<year>[0-9]{4})/$', views.yearly_archive, name='yearly_archive'),
]
```

### URL Reversing
Instead of hardcoding URLs in templates or views, dynamically resolve them.

```python
from django.urls import reverse
from django.shortcuts import redirect

def my_view(request):
    # reverse('app_namespace:url_name', args=[...], kwargs={...})
    url = reverse('blog:post_detail', kwargs={'post_id': 5})
    # url is now '/blog/post/5/'
    return redirect(url)
```

In templates:
```html
<a href="{% url 'blog:post_detail' post_id=5 %}">View Post</a>
```

💡 **Pro Tip:** Always use `app_name` and named URLs (`name="something"`). If your URLs change structure, you only update them in `urls.py`, and `reverse()` handles the rest.

⚠️ **Common Pitfalls:** Missing the trailing slash in `path('my-url/', ...)` or omitting `include()` when wiring up app URLs. Also, order matters! Django processes `urlpatterns` top-to-bottom and stops at the first match.

---

## 4. VIEWS

Views handle the business logic: receiving an `HttpRequest`, processing it, and returning an `HttpResponse`.

### Function-Based Views (FBVs)
FBVs are simple, explicit Python functions.

```python
# blog/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse, JsonResponse
from .models import Post

def post_list(request):
    # Retrieve all posts
    posts = Post.objects.all()
    # Context dictionary passed to the template
    context = {'posts': posts}
    # Render combines template and context into an HttpResponse
    return render(request, 'blog/post_list.html', context)

def api_posts(request):
    # Returning JSON data
    data = {'status': 'ok', 'count': 5}
    return JsonResponse(data)

def delete_post(request, pk):
    # Returns 404 if post doesn't exist
    post = get_object_or_404(Post, pk=pk)
    if request.method == 'POST':
        post.delete()
        # Redirect after successful POST
        return redirect('blog:post_list')
    return render(request, 'blog/post_confirm_delete.html', {'post': post})
```

### Class-Based Views (CBVs)
CBVs allow you to reuse code via inheritance and mixins. Django provides Generic CBVs for common tasks.

```python
from django.views.generic import (
    ListView, DetailView, CreateView, UpdateView, DeleteView, TemplateView
)
from django.urls import reverse_lazy
from .models import Post
from .forms import PostForm

class AboutView(TemplateView):
    # Simply renders a template
    template_name = 'blog/about.html'

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts' # default is object_list
    paginate_by = 10 # built-in pagination!

    def get_queryset(self):
        # Override to customize the query
        return Post.objects.filter(status='published')

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    # looks for a 'pk' or 'slug' in the URL by default

class PostCreateView(CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    # reverse_lazy is used because urls aren't loaded when class is defined
    success_url = reverse_lazy('blog:post_list')

    def form_valid(self, form):
        # Custom logic before saving
        form.instance.author = self.request.user
        return super().form_valid(form)
```

Wiring CBVs in `urls.py`:
```python
path('posts/', PostListView.as_view(), name='post_list'),
```

### FBV vs CBV Comparison
| Feature | Function-Based Views (FBVs) | Class-Based Views (CBVs) |
| :--- | :--- | :--- |
| **Simplicity** | High. Very explicit logic. | Lower. Relies on inherited behavior. |
| **Reusability** | Low. Must write decorator/helper functions. | High. Use mixins and inheritance. |
| **Readability** | High for simple logic, messy for complex logic. | Clean for CRUD, messy if overriding too many methods. |
| **Best For** | Custom, complex, unique logic. | Standard CRUD operations. |

### View Decorators
```python
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST

@login_required(login_url='/login/')
@require_POST
def like_post(request, pk):
    # Only authenticated users and POST requests allowed here
    pass
```
For CBVs, use `method_decorator` or Mixins like `LoginRequiredMixin`.

💡 **Pro Tip:** The Post/Redirect/Get (PRG) pattern prevents double-form submission. Always return a `redirect()` after a successful `POST` request.

⚠️ **Common Pitfalls:** Modifying class attributes directly in a CBV (e.g., `self.my_list.append()`) can share state across different user requests! Always modify instance variables inside methods like `get()` or `post()`, or `dispatch()`.

---

## 5. TEMPLATES

Django templates allow rendering dynamic content into HTML.

### Configuration
```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # Directories to search for templates globally
        'DIRS': [BASE_DIR / 'templates'], 
        'APP_DIRS': True, # Looks inside app/templates/ directories
        'OPTIONS': {
            'context_processors': [
                # Injects variables into all templates globally
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

### DTL (Django Template Language) Syntax
- **Variables:** `{{ variable_name }}`
- **Tags:** `{% logic %}`
- **Filters:** `{{ variable|filter }}`

### Template Inheritance
`base.html` (The Parent)
```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <nav>...</nav>
    <main>
        {% block content %}
        <!-- Child templates will inject code here -->
        {% endblock %}
    </main>
</body>
</html>
```

`child.html` (The Child)
```html
{% extends 'base.html' %}

{% block title %}About Us - {{ block.super }}{% endblock %}

{% block content %}
    <h1>About Us</h1>
    <p>We are a Django company.</p>
    
    {% include 'partials/contact_form.html' %}
{% endblock %}
```

### Tags and Filters Examples
```html
<!-- Loop -->
{% for post in posts %}
    <h2>{{ post.title|title }}</h2> <!-- Filter: title case -->
    <p>{{ post.body|truncatewords:30 }}</p>
    <p>Published on: {{ post.created_at|date:"F j, Y" }}</p>
{% empty %}
    <p>No posts available.</p>
{% endfor %}

<!-- If statements -->
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
{% else %}
    <p>Please log in.</p>
{% endif %}
```

### Custom Template Tags and Filters
Create a `templatetags` directory in your app with an `__init__.py`.
Create `blog_tags.py`:

```python
# blog/templatetags/blog_tags.py
from django import template
from ..models import Post

register = template.Library()

# Custom Filter
@register.filter(name='markdown_to_html')
def markdown_to_html(text):
    # e.g., use a markdown library to parse text
    return f"parsed: {text}"

# Custom Tag
@register.simple_tag
def total_posts():
    return Post.objects.count()
```
Usage in template:
```html
{% load blog_tags %}
<p>Total posts: {% total_posts %}</p>
<div>{{ post.content|markdown_to_html }}</div>
```

💡 **Pro Tip:** Never do heavy database queries inside templates. Prepare the data in the view or model layer, then pass it down. Templates should only handle presentation logic.

⚠️ **Common Pitfalls:** Forgetting to `{% load %}` custom tags or static tags at the top of the template file. Also, `{{ my_dict.key }}` is used for dict lookup, not `{{ my_dict['key'] }}`.

---

## 6. MODELS & ORM

The Object-Relational Mapper (ORM) maps Python classes to database tables.

### Model Definition
```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    
    class Meta:
        verbose_name_plural = 'Categories'
        ordering = ['name']

    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
    )

    # Database Fields
    title = models.CharField(max_length=200, db_index=True)
    slug = models.SlugField(max_length=200, unique=True)
    content = models.TextField(blank=True, null=True) # blank=form validation, null=DB level
    
    # Relationships
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='blog_posts')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    tags = models.ManyToManyField('Tag', blank=True)
    
    # Choices & Defaults
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True) # Sets on creation
    updated_at = models.DateTimeField(auto_now=True)     # Sets on every save

    class Meta:
        ordering = ['-created_at'] # Descending order
        # Compound indexing
        indexes = [
            models.Index(fields=['status', 'created_at']),
        ]

    def __str__(self):
        return self.title

class Tag(models.Model):
    name = models.CharField(max_length=50)
```

### Advanced Relationships
- **ForeignKey:** 1-to-Many. (e.g., Author to Posts). `on_delete=models.CASCADE` means if User is deleted, all their Posts are deleted.
- **OneToOneField:** 1-to-1. (e.g., User to UserProfile). Similar to FK with `unique=True`.
- **ManyToManyField:** Many-to-Many. (e.g., Post to Tags). Creates a hidden join table automatically.

### ORM QuerySets

```python
# Retrieving objects
all_posts = Post.objects.all()
first_post = Post.objects.first()
post_by_id = Post.objects.get(id=1) # Raises DoesNotExist if not found, MultipleObjectsReturned if >1

# Filtering
published_posts = Post.objects.filter(status='published')
exclude_drafts = Post.objects.exclude(status='draft')
chained = Post.objects.filter(status='published').exclude(author__username='admin')

# Field lookups (double underscore '__')
# exact, iexact, contains, icontains, gt, gte, lt, lte, startswith, in
recent_posts = Post.objects.filter(created_at__year=2024, title__icontains='django')
category_posts = Post.objects.filter(category__name='Technology') # Join!

# Creating and Updating
new_post = Post.objects.create(title='New Post', author=user)
Post.objects.filter(status='draft').update(status='published') # Bulk update

# Deleting
Post.objects.filter(title__contains='spam').delete()
```

### Advanced Querying (Q, F, Aggregation)
```python
from django.db.models import Q, F, Count, Sum, Avg

# Q Objects: For OR / Complex queries
Post.objects.filter(Q(title__icontains='Django') | Q(content__icontains='Python'))
Post.objects.filter(~Q(status='draft')) # NOT query

# F Expressions: Referencing fields at DB level without pulling into Python
Post.objects.filter(views__gt=F('likes')) # Where views > likes
Post.objects.update(views=F('views') + 1) # Atomic increment

# Aggregation & Annotation
# Aggregation calculates a single value
avg_views = Post.objects.aggregate(Avg('views')) 
# Annotation adds calculated data to EACH object
authors_with_counts = User.objects.annotate(num_posts=Count('blog_posts'))
for author in authors_with_counts:
    print(author.num_posts)
```

### Model Managers
You can create custom QuerySets and Managers to attach reusable queries to the model.
```python
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

class Post(models.Model):
    # ... fields
    objects = models.Manager() # The default manager
    published = PublishedManager() # Our custom manager

# Usage: Post.published.all()
```

### Database Transactions
Ensure atomic operations (all or nothing).
```python
from django.db import transaction

@transaction.atomic
def transfer_funds(from_acc, to_acc, amount):
    from_acc.balance -= amount
    from_acc.save()
    # If something breaks here, from_acc.balance rolls back!
    to_acc.balance += amount
    to_acc.save()
```

💡 **Pro Tip:** Always use `get_object_or_404()` in views instead of `get()` to prevent 500 Server Errors when a user enters a bad ID in the URL.

⚠️ **Common Pitfalls:** Modifying a QuerySet in a loop. e.g., `for post in posts: post.views += 1; post.save()`. This causes the N+1 query problem. Use `update()` instead: `posts.update(views=F('views') + 1)`.

---

## 7. MIGRATIONS

Migrations are Django's way of propagating changes you make to your models (adding a field, deleting a model, etc.) into your database schema.

### Core Commands
- `makemigrations`: Creates migration files based on model changes.
- `migrate`: Applies migrations to the database.
- `showmigrations`: Lists all migrations and their status.
- `sqlmigrate`: Shows the raw SQL for a specific migration.

```bash
python manage.py makemigrations blog
python manage.py sqlmigrate blog 0001
python manage.py migrate
```

### Data Migrations (RunPython)
Sometimes you need to migrate *data*, not just schema. You create an empty migration file:
`python manage.py makemigrations --empty blog`

```python
# blog/migrations/0002_custom_data.py
from django.db import migrations

def create_initial_data(apps, schema_editor):
    # IMPORTANT: Use apps.get_model, NOT direct import from models.py
    # This ensures you use the model state AT THIS POINT in migration history
    Category = apps.get_model('blog', 'Category')
    Category.objects.create(name='General')

def reverse_initial_data(apps, schema_editor):
    Category = apps.get_model('blog', 'Category')
    Category.objects.filter(name='General').delete()

class Migration(migrations.Migration):
    dependencies = [
        ('blog', '0001_initial'),
    ]

    operations = [
        migrations.RunPython(create_initial_data, reverse_initial_data),
    ]
```

💡 **Pro Tip:** Never manually delete migration files or rows from the `django_migrations` table unless you absolutely know what you are doing. If things get messed up, look into `squashmigrations` or `--fake`.

⚠️ **Common Pitfalls:** Adding a non-nullable field without a default to an existing model with data. Django will ask you for a one-off default during `makemigrations`. Choose wisely or database constraints will fail.

---

## 8. FORMS

Django forms handle HTML form generation, data validation, and conversion to Python types.

### Standard Forms
```python
# blog/forms.py
from django import forms
from django.core.exceptions import ValidationError

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100, widget=forms.TextInput(attrs={'class': 'form-control'}))
    email = forms.EmailField(required=True)
    message = forms.CharField(widget=forms.Textarea)
    
    # Custom Validation for a specific field
    def clean_email(self):
        email = self.cleaned_data.get('email')
        if not email.endswith('@company.com'):
            raise ValidationError('Please use a company email address.')
        return email

    # Custom Validation combining multiple fields
    def clean(self):
        cleaned_data = super().clean()
        name = cleaned_data.get('name')
        message = cleaned_data.get('message')
        if name and name.lower() in message.lower():
            raise ValidationError('Do not put your name in the message.')
        return cleaned_data
```

### ModelForms
Automatically generate forms tied to a Model.

```python
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'status']
        # exclude = ['author', 'created_at'] 
        widgets = {
            'status': forms.RadioSelect(),
        }
```

### Handling Forms in Views
```python
def create_post(request):
    if request.method == 'POST':
        # Bind the form to POST data
        form = PostForm(request.POST, request.FILES)
        if form.is_valid(): # Triggers clean() methods
            # Create instance but don't commit to DB yet
            post = form.save(commit=False)
            post.author = request.user # Attach missing data
            post.save()
            form.save_m2m() # Required if commit=False and using ManyToMany
            return redirect('blog:post_detail', post.id)
    else:
        # Unbound form for GET requests
        form = PostForm()
        
    return render(request, 'blog/post_form.html', {'form': form})
```

### Template Rendering
```html
<form method="post" enctype="multipart/form-data">
    <!-- CSRF token is MANDATORY for POST forms -->
    {% csrf_token %}
    
    <!-- Render form as paragraphs -->
    {{ form.as_p }}
    
    <button type="submit">Submit</button>
</form>
```

💡 **Pro Tip:** Use `django-crispy-forms` to quickly style forms using Bootstrap or Tailwind without writing manual HTML for every field.

⚠️ **Common Pitfalls:** Forgetting `request.FILES` when initializing a form that handles file/image uploads, or forgetting `enctype="multipart/form-data"` in the `<form>` HTML tag.

---

## 9. ADMIN

The built-in admin panel is one of Django's most powerful features.

### Registration and Customization
```python
# blog/admin.py
from django.contrib import admin
from .models import Post, Category, Tag

# Simple registration
admin.site.register(Category)
admin.site.register(Tag)

# Advanced registration via Decorator
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # Columns shown in the list view
    list_display = ('title', 'author', 'status', 'created_at', 'word_count')
    # Fields that act as links to edit the object
    list_display_links = ('title',)
    # Right sidebar filters
    list_filter = ('status', 'created_at', 'category')
    # Search box functionality
    search_fields = ('title', 'content', 'author__username')
    # Prepopulates the slug field via JS as you type the title
    prepopulated_fields = {'slug': ('title',)}
    # Makes fields read-only
    readonly_fields = ('created_at', 'updated_at')
    
    # Organizing the edit form layout
    fieldsets = (
        ('Content', {
            'fields': ('title', 'slug', 'content', 'tags')
        }),
        ('Meta Data', {
            'classes': ('collapse',), # Hides section by default
            'fields': ('author', 'category', 'status', 'created_at', 'updated_at')
        }),
    )

    # Custom column calculation
    def word_count(self, obj):
        return len(obj.content.split()) if obj.content else 0
    word_count.short_description = 'Words' # Column header name

    # Custom Action
    actions = ['make_published']

    @admin.action(description='Mark selected posts as published')
    def make_published(self, request, queryset):
        queryset.update(status='published')
        self.message_user(request, "Successfully published selected posts.")
```

### Inlines
Edit related models on the same page.
```python
class PostInline(admin.StackedInline): # or TabularInline
    model = Post
    extra = 1 # Number of empty forms to display

@admin.register(User) # Assuming we unregister the default User and re-register
class CustomUserAdmin(admin.ModelAdmin):
    inlines = [PostInline]
```

💡 **Pro Tip:** You can override admin templates completely. Create `templates/admin/base_site.html` in your project to quickly change the "Django administration" header text.

⚠️ **Common Pitfalls:** Putting heavy calculations in `list_display` functions. It will run for *every single row* displayed on the admin list page, killing performance. Use `get_queryset` to annotate values instead.

---

## 10. AUTHENTICATION & AUTHORIZATION

Django comes with a robust built-in auth system.

### Custom User Model
**CRITICAL:** Always set up a custom user model at the *start* of a new project, even if you don't need custom fields yet. Changing it later is extremely difficult.

```python
# users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    # Add extra fields
    bio = models.TextField(blank=True)
    is_premium = models.BooleanField(default=False)
```

```python
# settings.py
AUTH_USER_MODEL = 'users.CustomUser'
```

### Authentication Views & Decorators
Django provides built-in views for login, logout, password reset, etc.

```python
# urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='users/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

In your views:
```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
def dashboard(request):
    # request.user is guaranteed to be an authenticated User object here
    return render(request, 'dashboard.html')

@permission_required('blog.add_post', raise_exception=True)
def author_area(request):
    pass
```

### Manual Authentication
```python
from django.contrib.auth import authenticate, login, logout

def custom_login_view(request):
    # ... form validation ...
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user) # Sets session variables
        return redirect('home')
    else:
        # Return invalid credentials error
        pass
```

### Permissions and Groups
- Users can belong to Groups.
- Both Users and Groups can have Permissions.
- Permissions are auto-created for every model: `add`, `change`, `delete`, `view`.

```python
# Check permissions manually
if request.user.has_perm('blog.delete_post'):
    # Do something
```

💡 **Pro Tip:** Use `django-allauth` for easy integration of social authentication (Google, GitHub, Facebook) and comprehensive user registration flows (email verification).

⚠️ **Common Pitfalls:** Storing passwords in plaintext. Django handles password hashing (PBKDF2 by default). Never manually save a user's password using `user.password = '123'`, always use `user.set_password('123')`.

---

## 11. MIDDLEWARE

Middleware is a framework of hooks into Django's request/response processing. It's a light, low-level "plugin" system.

### Execution Order
Middleware runs sequentially top-to-bottom for the Request, and bottom-to-top for the Response.

### Writing Custom Middleware
```python
# core/middleware.py

class SimpleCustomMiddleware:
    def __init__(self, get_response):
        # One-time configuration and initialization.
        self.get_response = get_response

    def __call__(self, request):
        # 1. Code to be executed for each request before
        # the view (and later middleware) are called.
        print(f"Request path: {request.path}")
        
        # Add custom attributes to request
        request.custom_tracking_id = "12345"

        # 2. Call the next middleware or the view
        response = self.get_response(request)

        # 3. Code to be executed for each request/response after
        # the view is called.
        response['X-My-Custom-Header'] = "Hello"

        return response
    
    # Optional Hooks
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Called just before Django calls the view.
        pass

    def process_exception(self, request, exception):
        # Called when a view raises an exception.
        pass
```

### Registering Middleware
```python
# settings.py
MIDDLEWARE = [
    # ... built-ins ...
    'core.middleware.SimpleCustomMiddleware',
]
```

💡 **Pro Tip:** Middleware is synchronous by default. If you write Async views, wrap middleware in `sync_and_async_middleware` decorator to prevent context switching overhead.

⚠️ **Common Pitfalls:** Placing custom middleware at the wrong index in the `MIDDLEWARE` list. E.g., if you rely on `request.user`, your middleware must be placed *after* `AuthenticationMiddleware`.

---

## 12. STATIC & MEDIA FILES

### Static Files (CSS, JS, Images for the site design)
```python
# settings.py
STATIC_URL = '/static/' # URL prefix
STATICFILES_DIRS = [BASE_DIR / 'static'] # Where to look for global static files
STATIC_ROOT = BASE_DIR / 'staticfiles' # Where collectstatic dumps everything for production
```

In templates:
```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

### Media Files (User-uploaded content)
```python
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

In Models:
```python
class Profile(models.Model):
    # Uploads to media/avatars/
    avatar = models.ImageField(upload_to='avatars/') 
```

In Templates:
```html
<img src="{{ user.profile.avatar.url }}" alt="Avatar">
```

### Serving During Development
Django does NOT serve Media files by default in development. You must update `urls.py`:

```python
# core/urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... paths
]

if settings.DEBUG:
    # Serve media files in dev
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    # Static files are handled automatically by runserver in dev
```

💡 **Pro Tip:** In production, use `WhiteNoise` to let Django/Gunicorn serve static files efficiently, bypassing the need for complex Nginx static routing. For Media, use object storage like AWS S3 with `django-storages`.

⚠️ **Common Pitfalls:** Forgetting to run `python manage.py collectstatic` when deploying to production. This gathers all app-level and global static files into the `STATIC_ROOT` directory.

---

## 13. SESSIONS & COOKIES

### Sessions API
Sessions store data across requests for a specific browser. By default, Django stores session data in the database and sends a cookie (`sessionid`) to the browser containing a session key.

```python
def my_view(request):
    # Setting a session variable
    request.session['has_seen_popup'] = True
    
    # Getting a session variable
    has_seen = request.session.get('has_seen_popup', False)
    
    # Deleting a session variable
    if 'cart_items' in request.session:
        del request.session['cart_items']
        
    # Flush entire session (logout does this)
    # request.session.flush()
```

### Handling Cookies
```python
def set_cookie_view(request):
    response = HttpResponse("Cookie Set")
    # set_cookie(key, value, max_age, expires, path, domain, secure, httponly)
    response.set_cookie('theme', 'dark', max_age=3600, secure=True, httponly=True)
    return response

def get_cookie_view(request):
    theme = request.COOKIES.get('theme', 'light')
    return HttpResponse(f"Current theme: {theme}")
```

💡 **Pro Tip:** For performance, change the session engine to use Redis or Memcached instead of the Database if you have high traffic.
`SESSION_ENGINE = "django.contrib.sessions.backends.cache"`

⚠️ **Common Pitfalls:** Storing large objects or Model instances in sessions. Sessions are serialized (JSON by default in modern Django). Store the ID of the object instead of the whole object!

---

## 14. CACHING

Caching saves the result of an expensive calculation so it doesn't need to be computed on every request.

### Configuring Cache Backends
```python
# settings.py
CACHES = {
    'default': {
        # Using Redis as cache
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
```

### Caching Levels

**1. Per-View Caching:**
```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15) # Cache for 15 minutes
def heavy_view(request):
    return render(request, 'heavy.html')
```

**2. Template Fragment Caching:**
```html
{% load cache %}
<!-- Cache this specific HTML block for 500 seconds -->
{% cache 500 sidebar_cache_key %}
    {% include 'partials/expensive_sidebar.html' %}
{% endcache %}
```

**3. Low-Level Cache API:**
```python
from django.core.cache import cache

def get_expensive_data():
    data = cache.get('my_data_key')
    if not data:
        # Perform heavy DB query or API call
        data = perform_heavy_computation()
        cache.set('my_data_key', data, timeout=3600)
    return data
```

💡 **Pro Tip:** Cache invalidation is hard. Always define a strategy for clearing the cache when data changes. Use Django Signals to delete cache keys when models are updated.

⚠️ **Common Pitfalls:** Caching user-specific data (like a shopping cart) globally using `cache_page`. `cache_page` caches the view based on the URL; if it contains user-specific data, User A might see User B's data! Use the `Vary` header.

---

## 15. SIGNALS

Signals allow decoupled applications get notified when actions occur elsewhere in the framework.

### Built-in Signals
- `pre_save` / `post_save`: Before/After a model's `save()` method.
- `pre_delete` / `post_delete`: Before/After a model's `delete()` method.
- `m2m_changed`: When a ManyToMany field is modified.

### Creating and Connecting Signals
Create a `signals.py` in your app.

```python
# users/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

# The receiver decorator connects the function to the signal
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    # 'instance' is the User being saved
    # 'created' is True if this is a new record
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### Loading Signals
Signals won't run unless imported. Import them in the app's `apps.py`.

```python
# users/apps.py
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'users'

    def ready(self):
        # Import signals when the app is ready
        import users.signals
```

💡 **Pro Tip:** Use signals for things that are completely decoupled, like sending a welcome email or creating an initial profile.

⚠️ **Common Pitfalls:** Putting business logic inside signals. It makes code very hard to trace and debug ("spooky action at a distance"). If the logic is intrinsically tied to the model, override the model's `save()` method instead.

---

## 16. REST APIs WITH DJANGO REST FRAMEWORK (DRF)

DRF is the standard for building Web APIs in Django.

### Installation
`pip install djangorestframework` and add `rest_framework` to `INSTALLED_APPS`.

### Serializers
Convert Model instances to JSON and validate incoming JSON data.

```python
# api/serializers.py
from rest_framework import serializers
from blog.models import Post
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class PostSerializer(serializers.ModelSerializer):
    # Nested Serialization
    author = UserSerializer(read_only=True)
    # Custom computed field
    slug_length = serializers.SerializerMethodField()

    class Meta:
        model = Post
        fields = '__all__'
    
    def get_slug_length(self, obj):
        return len(obj.slug)
```

### ViewSets & Routers
ViewSets combine the logic for related views (List, Retrieve, Create, Update, Destroy) into a single class.

```python
# api/views.py
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from rest_framework.decorators import action
from rest_framework.response import Response
from .serializers import PostSerializer
from blog.models import Post

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]

    # Custom API endpoint: /api/posts/<id>/publish/
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.status = 'published'
        post.save()
        return Response({'status': 'post published'})
```

```python
# api/urls.py
from rest_framework.routers import DefaultRouter
from django.urls import path, include
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

### DRF Configuration
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

💡 **Pro Tip:** For JWT (JSON Web Tokens), use the `djangorestframework-simplejwt` package.

⚠️ **Common Pitfalls:** DRF also suffers from the N+1 problem! If a Serializer has nested relationships, make sure you override `get_queryset()` in the ViewSet to use `select_related()` and `prefetch_related()`.

---

## 17. TESTING

Django has a built-in test framework based on Python's `unittest`.

### Writing Tests
```python
# blog/tests.py
from django.test import TestCase, Client
from django.urls import reverse
from .models import Post
from django.contrib.auth.models import User

class PostModelTest(TestCase):
    def setUp(self):
        # Runs before every test method
        self.user = User.objects.create_user(username='testuser', password='123')
        self.post = Post.objects.create(title='Test Post', slug='test-post', author=self.user)

    def test_post_creation(self):
        # Tests logic
        self.assertEqual(self.post.title, 'Test Post')
        self.assertTrue(isinstance(self.post, Post))
        self.assertEqual(str(self.post), 'Test Post')

class PostViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.url = reverse('blog:post_list')

    def test_post_list_view_status_code(self):
        response = self.client.get(self.url)
        self.assertEqual(response.status_code, 200)

    def test_post_list_view_template(self):
        response = self.client.get(self.url)
        self.assertTemplateUsed(response, 'blog/post_list.html')
```

### Running Tests
`python manage.py test`

💡 **Pro Tip:** Use `pytest` and `pytest-django` instead of the standard `TestCase` for less boilerplate, easier fixtures, and much faster test execution.

⚠️ **Common Pitfalls:** Testing code that connects to external APIs without mocking them. Always use Python's `unittest.mock.patch` to mock external network requests so your tests remain fast, deterministic, and offline capable.

---

## 18. SECURITY

Django provides excellent protection against common vulnerabilities.

### Built-in Protections
- **CSRF (Cross-Site Request Forgery):** `CsrfViewMiddleware` requires `{% csrf_token %}` in all POST forms.
- **XSS (Cross-Site Scripting):** Templates auto-escape all variables by default. To disable intentionally: `{{ var|safe }}`.
- **SQL Injection:** The ORM parameterizes all queries. Raw SQL via `RawSQL` or `cursor.execute` must be handled carefully.
- **Clickjacking:** `XFrameOptionsMiddleware` prevents site from being loaded in an `<iframe>`.

### Production Security Checklist
```python
# settings.py for Production

# Ensure DEBUG is absolutely False
DEBUG = False

# Must be set when DEBUG is False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Redirect HTTP to HTTPS
SECURE_SSL_REDIRECT = True

# Secure session and CSRF cookies
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# HSTS (HTTP Strict Transport Security)
SECURE_HSTS_SECONDS = 31536000 # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
```

Run `python manage.py check --deploy` to verify security settings.

💡 **Pro Tip:** Use `django-csp` to implement a Content Security Policy (CSP), which heavily mitigates XSS by dictating exactly which domains can load scripts/styles on your page.

⚠️ **Common Pitfalls:** Using `mark_safe()` or the `|safe` template filter on raw user input. This entirely bypasses XSS protections. Always sanitize input if you must render raw HTML (e.g., using `bleach`).

---

## 19. DEPLOYMENT

### WSGI vs ASGI
- **WSGI (Web Server Gateway Interface):** Standard sync Python deployment. Uses `Gunicorn` or `uWSGI`.
- **ASGI (Asynchronous Server Gateway Interface):** Async deployment. Uses `Uvicorn` or `Daphne`.

### Production Stack Example
1. **Nginx:** Reverse Proxy, serves static media, handles SSL.
2. **Gunicorn:** Application Server, runs the Django code via WSGI.
3. **PostgreSQL:** Production Database.
4. **Redis:** Cache and Celery message broker.

### Gunicorn Setup
`pip install gunicorn`
`gunicorn core.wsgi:application --bind 0.0.0.0:8000 --workers 3`

### Environment Configuration
Use `django-environ`:
```python
# settings.py
import environ
import os

env = environ.Env()
# Read .env file
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))

SECRET_KEY = env('SECRET_KEY')
DEBUG = env.bool('DEBUG', default=False)
DATABASES = {
    # Parses strings like postgres://user:pass@localhost/dbname
    'default': env.db(), 
}
```

💡 **Pro Tip:** Automate your deployments using Docker and CI/CD pipelines (GitHub Actions). Containerizing Django prevents "it works on my machine" issues.

⚠️ **Common Pitfalls:** Storing SQLite database in a Docker container or ephemeral server (like Heroku) without mounting a persistent volume. The database will wipe on every restart! Always use a managed PostgreSQL DB.

---

## 20. ASYNC DJANGO

Since Django 3.1, async views and middleware are supported. Since 4.1, the ORM supports async.

### Async Views
```python
import asyncio
from django.http import JsonResponse

async def my_async_view(request):
    # Simulating a non-blocking I/O call
    await asyncio.sleep(1)
    return JsonResponse({'message': 'Hello Async World'})
```

### Async ORM
Using the `a` prefix for ORM methods.
```python
async def get_user_data(request):
    # Async DB query
    user = await User.objects.aget(username='admin')
    # Async filtering
    async for post in Post.objects.filter(author=user):
        print(post.title)
        
    return JsonResponse({'status': 'done'})
```

### Sync/Async Transitions
If you need to call Sync code in an Async view (or vice versa):
```python
from asgiref.sync import sync_to_async, async_to_sync

def expensive_sync_function():
    # some old sync library
    pass

async def async_view(request):
    # Run sync function in threadpool
    result = await sync_to_async(expensive_sync_function)()
    return JsonResponse({'result': result})
```

💡 **Pro Tip:** For real-time features like WebSockets (chat apps, live notifications), you must use **Django Channels**, which wraps around ASGI to handle WebSocket protocols.

⚠️ **Common Pitfalls:** Mixing Sync and Async improperly. If you run a heavy Sync blocking operation inside an Async view without `sync_to_async`, you will block the entire async event loop, killing concurrency.

---

## 21. PERFORMANCE OPTIMIZATION

### The N+1 Query Problem
The most common Django performance killer.
```python
# BAD: 1 query for posts + N queries for each author!
posts = Post.objects.all()
for p in posts:
    print(p.author.username) 

# GOOD: Uses select_related (for ForeignKeys) - 1 SQL JOIN query total
posts = Post.objects.select_related('author').all()
for p in posts:
    print(p.author.username)

# GOOD: Uses prefetch_related (for ManyToMany / Reverse ForeignKeys) - 2 queries total
posts = Post.objects.prefetch_related('tags').all()
for p in posts:
    print([tag.name for tag in p.tags.all()])
```

### Database Indexing
Add `db_index=True` to fields you filter by frequently.
```python
class Post(models.Model):
    slug = models.SlugField(db_index=True)
```

### Query Optimization
- Only select fields you need: `Post.objects.values('id', 'title')`
- Defer heavy fields (like TextFields) if not needed immediately: `Post.objects.defer('content')`
- Check if items exist without loading them: `if Post.objects.filter(status='draft').exists():`
- Count without loading: `Post.objects.count()`

💡 **Pro Tip:** Install `django-debug-toolbar` in development. It intercepts requests and shows a sidebar detailing EXACTLY how many SQL queries ran, their execution time, and highlights duplicated queries.

⚠️ **Common Pitfalls:** Using `len(queryset)` instead of `queryset.count()`. `len()` evaluates the queryset and loads ALL objects into Python memory just to count them!

---

## 22. ADVANCED TOPICS

### Custom Management Commands
Write scripts that hook into `manage.py`.
```python
# app/management/commands/close_polls.py
from django.core.management.base import BaseCommand
from polls.models import Poll

class Command(BaseCommand):
    help = 'Closes the specified poll for voting'

    def add_arguments(self, parser):
        parser.add_argument('poll_ids', nargs='+', type=int)

    def handle(self, *args, **options):
        for poll_id in options['poll_ids']:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                self.stderr.write(self.style.ERROR(f'Poll {poll_id} does not exist'))
                continue
            poll.opened = False
            poll.save()
            self.stdout.write(self.style.SUCCESS(f'Successfully closed poll {poll_id}'))
```
Usage: `python manage.py close_polls 1 2 3`

### Celery Background Tasks
For long-running tasks (emails, data processing) so you don't block the HTTP response.
```python
# tasks.py
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def send_welcome_email_task(user_email):
    send_mail('Welcome', 'Thanks for joining!', 'admin@site.com', [user_email])

# In views.py:
def signup(request):
    # ...
    send_welcome_email_task.delay(user.email) # Runs in background worker
    return redirect('home')
```

### Signals of Awesome Third-Party Packages
- **django-extensions:** Adds `shell_plus` (auto-imports all models) and `RunScript`.
- **django-filter:** Powerful URL parameter filtering for QuerySets and DRF.
- **Pillow:** Required for Django's `ImageField` processing.
- **django-cors-headers:** Required if making API requests from a different frontend domain (React/Vue).

💡 **Pro Tip:** Keep an eye on `django-htmx`. Htmx allows you to build modern, dynamic SPA-like behaviors (live search, lazy loading) using pure HTML attributes and standard Django HTML responses, bypassing complex JS frameworks!

⚠️ **Common Pitfalls:** Over-engineering. Django is built on a "monolith first" philosophy. Don't split your Django app into microservices or headless APIs with separate frontends unless the project scale explicitly demands it. A well-built Django monolith can easily serve millions of requests.

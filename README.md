#### models:

```
class Category(models.Model):
    name = models.CharField(max_length=255, unique=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = 'Category'
        verbose_name_plural = 'Categories'


class Tag(models.Model):
    name = models.CharField(max_length=255, unique=True)

    def __str__(self):
        return self.name


class Blog(models.Model):
    title    = models.CharField(max_length=255)
    slug     = models.SlugField(blank=True, null=True, unique=True)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, blank=True, null=True)
    body     = models.TextField(blank=True, null=True)
    tags     = models.ManyToManyField(Tag, blank=True)
    author   = models.ForeignKey(User, default=1, on_delete=models.SET_DEFAULT)
    created  = models.DateTimeField(auto_now=False, auto_now_add=True)
    updated  = models.DateTimeField(auto_now=True, auto_now_add=False)

    def __str__(self):
        return self.title


class Communities(models.Model):
    name       = models.CharField(max_length=255)
    blog_posts = models.ManyToManyField(Blog, blank=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = 'Community'
        verbose_name_plural = 'Communities'
```
---
#### without select_related:

```
@query_debugger
def bld():

    qs = Blog.objects.all()

    print(qs.query)

    posts = []
    for item in qs:
        posts.append({
            'id': item.id,
            'title': item.title,
            'slug': item.slug,
            'category': item.category,
            'body': item.body,
            'tags': item.tags,
            'author': item.author,
            'created': item.created,
            'updated': item.updated,
        })

    return posts

```
```
>>> bld()
SELECT "blog_blog"."id", "blog_blog"."title", "blog_blog"."slug", "blog_blog"."category_id", "blog_blog"."body", "blog_blog"."author_id", "blog_blog"."created", "blog_blog"."updated" FROM "blog_blog"
Queries quantity: 2001
Execution time: 1.21s
```
---
#### select_related v1:

```
@query_debugger
def bld():

    qs = Blog.objects.select_related('category')

    print(qs.query)

    posts = []
    for item in qs:
        posts.append({
            'id': item.id,
            'title': item.title,
            'slug': item.slug,
            'category': item.category,
            'body': item.body,
            'tags': item.tags,
            'author': item.author,
            'created': item.created,
            'updated': item.updated,
        })

    return posts

```
```
>>> bld()
SELECT "blog_blog"."id", "blog_blog"."title", "blog_blog"."slug", "blog_blog"."category_id", "blog_blog"."body", "blog_blog"."author_id", "blog_blog"."created", "blog_blog"."updated", "blog_category"."id", "blog_category"."name" FROM "blog_blog" LEFT OUTER JOIN "blog_category" ON ("blog_blog"."category_id" = "blog_category"."id")
Queries quantity: 1001
Execution time: 0.78s

```
---
#### select_related v2:

```
@query_debugger
def bld():

    qs = Blog.objects.select_related('category', 'author')

    print(qs.query)

    posts = []
    for item in qs:
        posts.append({
            'id': item.id,
            'title': item.title,
            'slug': item.slug,
            'category': item.category,
            'body': item.body,
            'tags': item.tags,
            'author': item.author,
            'created': item.created,
            'updated': item.updated,
        })

    return posts

```
```
>>> bld()
SELECT "blog_blog"."id", "blog_blog"."title", "blog_blog"."slug", "blog_blog"."category_id", "blog_blog"."body", "blog_blog"."author_id", "blog_blog"."created", "blog_blog"."updated", "blog_category"."id", "blog_category"."name", "auth_user"."id", "auth_user"."password", "auth_user"."last_login", "auth_user"."is_superuser", "auth_user"."username", "auth_user"."first_name", "auth_user"."last_name", "auth_user"."email", "auth_user"."is_staff", "auth_user"."is_active", "auth_user"."date_joined" FROM "blog_blog" LEFT OUTER JOIN "blog_category" ON ("blog_blog"."category_id" = "blog_category"."id") INNER JOIN "auth_user" ON ("blog_blog"."author_id" = "auth_user"."id")
Queries quantity: 1
Execution time: 0.18s

```

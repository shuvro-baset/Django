resource : https://www.youtube.com/watch?v=iQF6pln3Gog&list=PLOLrQ9Pn6cazjoDEnwzcdWWf4SNS0QZml

ORM - Object relational mapper

Student.objects.all() = Select * from Student

* Why ORM? 
    - Write in the language you are already using anyway
    - Easy switching from MySql to PostgreSQL
    - ORM provides additional features out of the box
    - Queries can perform better - if you're not too familiar with Sql.
    - SQL provides us the ability to fine tune queries
        - ORM - potential for reduced performance
    - Impedance Mismatch
        - moving data from objecs to and from relational tables. 

`Simple OR queries`
- simple OR query
- Q objects - OR query
- query performance 

* check sql query and runtime -> objects.query, connectin.queries(import django.db connectin)

* OR query using (|) => posts = Student.objects.filter(surname__startswith='austin') | Student.objects.filter(surname__startswith='baldwin')
* OR query using (Q) => posts = Student.objects.filter(Q(surname__startswith='austin') | ~Q (surname__startswith='baldwin') | Q (surname__startswith='avery-parker'))

`AND Queries`
*   posts = Student.objects.filter(classroom=1) & Student.objects.filter(age=20)
*   posts = Student.objects.filter(Q(surname__startswith='baldwin')&Q(firstname__startswith='lakisha'))

`UNION Queries`
*  posts = Student.objects.all().values_list("firstname").union(Teacher.objects.all().values_list("firstname"))
note: 
    It will join two table based on query. and if column name same then it will show one row. 

    if we write values rather than values_list it will show a dictionary value.

`NOT Queries`
* posts = Student.objects.exclude(age__gt=19)
* posts = Student.objects.filter(~Q(age__gt=20)&~Q(surname__startswith='baldwin'))
* ~Q = NOT

`SELECT & OUTPUT INDIVIDUAL FILEDS`
posts = Student.objects.filter(classroom=1).only('firstname','age')

`Simple PERFORMING RAW QUERIES`
* sql = "SELECT * FROM student_student"
    posts = Student.objects.raw(sql)[:2]


################
# Simple BYPASSING ORM
################

def dictfetchall(cursor): 
    desc = cursor.description 
    return [
            dict(zip([col[0] for col in desc], row)) 
            for row in cursor.fetchall() 
    ]

def student_list(request):

    cursor = connection.cursor()
    cursor.execute("SELECT * FROM student_student WHERE age > 20")
    r = dictfetchall(cursor)

    print(connection.queries)

    return render(request,'output.html',{'data': r})

`Model Inheritance`
from django.db import models
from django.utils import timezone

# Abstract Model

class BaseItem(models.Model):
    title = models.CharField(max_length=255)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
        ordering = ['title']

    def __str__(self):
        return self.title

class ItemA(BaseItem):
    content = models.TextField()

    class Meta(BaseItem.Meta):
        ordering = ['-created']

class ItemB(BaseItem):
    file = models.FileField(upload_to='files')

class ItemC(BaseItem):
    file = models.FileField(upload_to='images')

class ItemD(BaseItem):
    slug = models.SlugField(max_length=255, unique=True)


# Multi-table model inheritance

class Books(models.Model):
    title = models.CharField(max_length=100)
    created = models.DateTimeField(auto_now_add=True)

class ISBN(Books):
    books_ptr = models.OneToOneField(
        Books, on_delete=models.CASCADE,
        parent_link=True,
        primary_key=True,
    )
    ISBN = models.TextField()


# Proxy Model

class NewManager(models.Manager):
    pass

class BookContent(models.Model):
    title = models.CharField(max_length=100)
    created = models.DateTimeField(auto_now_add=True)

class BookOrders(BookContent):
    objects = NewManager()
    class Meta:
        proxy = True
        ordering = ['created']

    def created_on(self):
        return timezone.now() - self.created


`Django debug toolbar`

=========================== 
# getting data from two table that inherit from parent table
products = Product.objects.all().select_related('books', 'cupboard')
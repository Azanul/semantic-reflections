+++
title = "Django ORM's Hidden Magic: Advanced Uses and Techniques, Part-2"
date = 2023-02-02
[taxonomies]
tags = ["python", "django"]
+++

Welcome back for the 2nd part, check out the first one too, as we'll be using the same models we defined there. If you're familiar with `Question` and `Choice` models in Django's official documentation that'll work too.  
Let's continue

## Relationships

Relationships can be complicated but they don't have to be. In Django, relationships between models are defined using fields such as `ForeignKey`, `ManyToManyField`, and `OneToOneField`. These fields establish connections between tables in the database, allowing for related data to be retrieved with a single query. Relationships can be defined in two ways: one-to-many, where one model instance is related to multiple instances of another model, or many-to-many, where many instances of one model are related to many instances of another model. We've been using `ForeignKey` to link `Question` to `Choice` . Let's make some changes

```python
class QuestionType(models.Model):
    name = models.CharField(max_length=100)

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    question_type = models.OneToOneField(QuestionType, on_delete=models.CASCADE)
    choices = models.ManyToManyField(Choice)

class Choice(models.Model):
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

`OneToOneField` & `ManyToManyField` are both self-explanatory. One big upside of these relationships, besides ensuring proper classification of data, is that we can prefetch related data if we knew we're going to need it. Let's look into how we can do that.

## Prefetch

Prefetching in Django ORM allows you to fetch related objects in bulk, reducing the number of database queries required to retrieve data. This is particularly useful when dealing with complex relationships or when you need to retrieve data from multiple related objects.

```python
# Fetch all questions with related choice objects
questions = Question.objects.all().prefetch_related('choice_set')

# Accessing the related choice objects won't result in another query
for question in questions:
    print(question.question_text)
    for choice in question.choice_set.all():
        print(f" - {choice.choice_text}: {choice.votes} votes")
```

There are some dangers of prefetch that we need to be wary of. It adds overhead and increases memory consumption.

## F expressions

Django ORM's f expressions provide a way to perform database operations and calculate values on the fly without accessing the database multiple times. The f expression allows you to define custom computations within the query and avoids the need for manual calculations outside of the database.  
One of the main advantages of using f expressions is that they can be used in queries, aggregations, and updates, providing a flexible and efficient way to perform complex database operations.

```python
from django.db.models import F

# Calculate the percentage of votes each choice has received compared to the total number of votes
question = Question.objects.get(id=1)
choices = Choice.objects.filter(question=question).annotate(percentage=100 * F('votes') / F('question__choice__votes__sum'))
```

In this example, we first retrieve a specific question using the `Question.objects.get` method. Then, we filter the related `Choice` objects for that question and annotate each choice with a new field, `percentage`, which represents the percentage of votes that each choice has received. This percentage is calculated by dividing the number of votes for each choice (`votes`) by the total number of votes for all choices (`question__choice__votes__sum`). The `F` expression allows us to perform these calculations directly in the database and return the results as a queryset, rather than having to perform the calculations in Python code.

## Transactions

A transaction is started automatically when the first database operation is executed. All database operations executed within a transaction are committed together or rolled back if an error occurs. By default, Django automatically starts a new transaction for every request-response cycle, ensuring that any database changes made during a request are not persisted if an error occurs.

To explicitly control transactions in your Django application, you can use the `transaction.atomic` decorator or context manager. Within a transaction block, you can execute any number of database operations, and either commit the transaction by returning normally or roll back the transaction by raising an exception.

Let's try first with `transaction.atomic` decorator.

```python
from django.db import transaction

@transaction.atomic
def my_view(request):
    # database operations here
    # ...
    # if no exceptions, the transaction will be committed
    # if an exception is raised, the transaction will be rolled back
```

Now using a context manager

```python
from django.db import transaction

def my_view(request):
    with transaction.atomic():
        # database operations here
        # ...
        # if no exceptions, the transaction will be committed
        # if an exception is raised, the transaction will be rolled back
```

## Indexing

In order to optimize database queries, we can add indexes to specific fields in our models. This is particularly useful for columns that are frequently used for filtering, sorting or grouping data. In Django, indexes can be created either through the database directly or through Django models. Let's look at an example, if we want to search questions by `pub_date`, we

```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published', db_index=True)
```

we'll get in deeper into DB indexing in a future article, for now, that's all you need.

## Caching

Django provides a cache framework that allows you to cache your entire site or specific parts of it. To use caching in Django, you need to add a cache backend to your settings file. For example, you can use `memcached` as a cache backend:

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
```

Then, you can cache specific views or parts of your site using the `cache` decorator or the `cache` template tag. For example, to cache the result of a view for 60 \* 15 seconds (15 minutes):

```python
from django.shortcuts import render
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def my_view(request):
    ...
    return render(request, 'my_template.html', context)
```

Django official documentation is quite good for this section, definitely check it out if you're looking in caching. Here's the [link](https://docs.djangoproject.com/en/4.1/topics/cache/).

## Conclusion

In conclusion, Django ORM offers a range of powerful tools to help you manage your database efficiently. From custom lookups, aggregations, and raw SQL queries to model inheritance, proxy models, relationships, and efficient caching techniques, the Django ORM provides a comprehensive and flexible framework for database management.

By mastering these advanced techniques, you can streamline your database operations, improve the performance of your applications, and take full advantage of the power and scalability of Django. Whether you are a seasoned Django developer or just getting started, I hope this article has given you valuable insights into the hidden magic of Django ORM and how to use it to its full potential.

To get notified for cool articles like this in the future, follow me on Twitter at [**AzanulZ**](https://twitter.com/AzanulZ) or connect with me on LinkedIn, [**Azanul Haque**](https://www.linkedin.com/in/azanul-haque/).

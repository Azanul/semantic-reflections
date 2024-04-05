---
layout: post
title:  "Django ORM's Hidden Magic: Advanced Uses and Techniques, Part-1"
date:   2023-02-02 17:05:00 +0530
categories: python django
---

I'm assuming you already know what Django is and what is ORM. If not:-  
Django is a high-level web framework for building web applications quickly and with less code. It is written in Python and follows the Model-View-Template (MVT) architectural pattern. Django provides many built-in features such as a user authentication system, admin interface, and database abstraction layer, which makes it easier to develop web applications.

An important part of the database abstraction layer is ORM, which stands for Object-Relational Mapping. ORM is a feature of Django that allows you to interact with your database as if you were working with Python objects. Instead of writing raw SQL queries, you can use the Django ORM to perform database operations in a more intuitive and Pythonic way. The ORM provides a high-level API for creating, retrieving, updating, and deleting records in your database. It also supports complex operations such as aggregations, relationships, and custom lookups, allowing you to write efficient and scalable database code for your Django projects.

We'll be using the official Django polls app example.

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Now let's jump into the cool stuff:

## Custom Lookups

We can define custom lookups to perform complex database queries. For example, we can create a custom lookup to search for questions that lie within a certain length range. You can define the lookup as:

```python
from django.db.models import Field, Lookup

@Field.register_lookup
class LengthWithinLookup(Lookup):
    lookup_name = 'length_within'

    def as_sql(self, compiler, connection):
        lhs, lhs_params = self.process_lhs(compiler, connection)
        rhs, rhs_params = self.process_rhs(compiler, connection)
        lower_limit, upper_limit = rhs_params[0][1:-1].split(", ")
        return f"LENGTH({lhs}) BETWEEN {lower_limit} AND {upper_limit}", []

...
```

We can query questions with length within the range \[16, 40\] by running:

```python
Question.objects.filter(question_text__length_within=(16, 40))
```

For more info on custom lookups visit [this link](https://docs.djangoproject.com/en/4.1/howto/custom-lookups/).

## Aggregation & Annotation

Django ORM provides built-in support for database-level aggregation operations, such as counting, summing, averaging, grouping and more. These operations can be performed using the `django.db.models` API. The `aggregate` and `annotate` methods allow for powerful data analysis and manipulation and can help you write cleaner, more efficient code. Let's see a few examples:

```python
# Total number of questions.
Question.objects.count()

# Max votes across all choices.
from django.db.models import Max
Choice.objects.aggregate(Max('votes'))

# Annotate each question with the no. of choices for it
Question.objects.annotate(num_choices=Count('choice'))

# Annotate each question with the choice_text of choice with highest votes
from django.db.models import Subquery, OuterRef
Question.objects.annotate(
    choice_text=Subquery(
        Choice.objects.filter(question=OuterRef('pk'))
        .order_by('-votes')
        .values('choice_text')[:1]
    )
)
```

I recommend that you take a look at the official documentation if you think you need either aggregation & annotation as it's much more detailed there, here's the [link](https://docs.djangoproject.com/en/4.1/topics/db/aggregation/). Feel free to comment below if you have any queries not covered there. We need to be careful about yielding wrong results while aggregating & annotating, check out this [link](https://code.djangoproject.com/ticket/10060). In the last example we've used a subquery, let's look further into it.

## Subqueries

Subqueries in Django ORM allow you to use the results of one query as input for another query. You can perform complex operations by chaining multiple queries together, which can lead to more efficient and readable code.

Subqueries can be used in several ways in the Django ORM, including as a filter, as an annotation, or as an expression. To use a subquery as a filter, you can use the Subquery expression in the filter method. To use a subquery as an annotation, you can use the Subquery expression in the annotate method. To use a subquery as an expression, you can use it as an input to other expressions, such as F expressions (we'll look into these in the next part). Lets take an example, let's say you want to find all questions that have at least one choice with a vote count greater than 5. One way to achieve this would be to use the `exclude` method along with a subquery:

```python
from django.db.models import Subquery, OuterRef

question_with_high_votes = Choice.objects.filter(
    votes__gt=5, question=OuterRef('pk')
).values('question')

Question.objects.exclude(
    pk__in=Subquery(question_with_high_votes)
)
```

reference [link](https://docs.djangoproject.com/en/4.1/ref/models/expressions/#subquery-expressions).

## Model Inheritance

This section can be an entire article all by itself and we still won't be able to cover it all. I hope you're already familiar with OOP. Django ORM supports several forms of model inheritance. The main forms are:

1. Abstract Base Classes: An abstract base class is a model that is meant to be used as a base class for other models, but it won't be instantiated itself. It provides fields that can be reused by other models but it is not stored in the database. Let's update our `Question` and `Choice` models to have `created_at` and `updated_at` times. We'll do so by making both of them extend an abstract class `BaseModel`.
    
    ```python
    # Abstract class
    class BaseModel(models.Model):
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
    
        class Meta:
            abstract = True
    
    class Question(BaseModel):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')
    
    class Choice(BaseModel):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)
    ```
    
    Both `Question` and `Choice` will have all their fields and those of `BaseModel` .
    
2. Multi-table Inheritance: Multi-table inheritance allows a model to inherit fields and methods from multiple parent models. The fields and methods of the parent models are stored in separate tables in the database, with the child model having its table for its fields. Let's extend our example for this:
    
    ```python
    class BaseModel(models.Model):
        created_at = models.DateTimeField(auto_now_add=True)
        updated_at = models.DateTimeField(auto_now=True)
    
        class Meta:
            abstract = True
    
    class CommunityModel(models.Model):
        author = models.CharField(max_length=100)
    
    class Question(BaseModel):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')
    
    class Choice(BaseModel):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)
    
    class CommunityChoice(Choice, CommunityModel):
        pass
    ```
    
    `CommunityChoice` will have all fields from `Choice` as well as from `CommunityModel` .
    
3. Proxy models: Proxy models are models that inherit from an existing model, but don't add any fields or methods. They are typically used to change the manager for a model, or change the default ordering of the model's objects.
    
    ```python
    class PublishedQuestion(Question):
        class Meta:
            proxy = True
    
        def was_published_recently(self):
            return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
    ```
    
    In this example, `PublishedQuestion` is a proxy model that extends the `Question` model. The `PublishedQuestion` model has a method `was_published_recently` that checks if the question was published on the last day. Because `PublishedQuestion` is a proxy model, it has the same fields and behavior as the `Question` model, but can have additional methods and attributes.
    

Let me clear up any dangling confusion among these three. Abstract models do not create DB tables, neither do Proxy models. Proxy models inherit a solid (has a DB table) model class, Abstract models are inherited for adding extra fields.

## Raw SQL Query

If nothing else works for you, but you know SQL then you can run raw SQL queries directly to the connected database. We don't even have to create a new connection. We use the same persistent connection created by Django. Let's fetch all questions published before 1st Feb 2023.

```python
from django.db import connection


cursor = connection.cursor()
cursor.execute("SELECT * FROM polls_question WHERE pub_date < %s", ['2023-02-01'])

result = cursor.fetchall()

cursor.close()
```

But what if we wanted `Question` objects instead of raw query results. We can get it like this:

```python
Question.objects.raw("SELECT * FROM polls_question WHERE pub_date < %s", ['2023-02-01'])
```

We can even fetch question data from a different table, all we need is to provide the mapping from the source fields to destination fields if fields are differently numbered or named. Let's take a separate table

```python
class NotQuestion(models.Model):
    question_text = models.CharField(max_length=200)
    due_date = models.DateTimeField('due date')
    category = models.CharField(max_length=100)
```

Now there are still two ways to do this:

```python
field_map = {'due_date': 'pub_date'}
Question.objects.raw('SELECT * FROM polls_notquestion', translations=field_map)
```

This way we get the fields from `NotQuestion` in `Question` objects. The other way is to use the `AS` syntax in SQL.

```python
Question.objects.raw('SELECT question_text, due_date AS pub_date FROM polls_notquestion')
```

## Conclusion

Soon enough I'll add the next part to it, in which we'll cover Relationships, Prefetch, F expressions, Transactions and Indexing & Caching. To get notified when it releases, follow me on Twitter at [AzanulZ](https://twitter.com/AzanulZ) or connect with me on LinkedIn, [Azanul Haque](https://www.linkedin.com/in/azanul-haque/).

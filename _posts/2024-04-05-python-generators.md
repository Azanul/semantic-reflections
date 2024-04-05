---
layout: post
title:  "Python generators"
date:   2024-04-04 17:05:00 +0530
categories: python advanced
---

[Generators] are a powerful concept in Python that offer memory-efficient ways to create iterators. Unlike traditional iterables like lists or tuples that store all values in memory at once, [generators] produce values on demand, making them ideal for handling large datasets, data processing pipelines or infinite sequences.

Let's take an example to dive into [generators]. Imagine that you're a chef and you don't want to prepare the ingredients beforehand as you don't know how many and which dishes are going to sell today. We're going to create a pipeline from ordering random dishes to preparing and serving them.

1. Define a menu and the ingredients required for each dish first
  ```python
  ingredients = {
      "Pizza": ["Flour", "Yeast", "Tomato Sauce", "Mozzarella Cheese", "Pepperoni"],
      "Spaghetti": ["Pasta", "Ground Beef", "Tomato Sauce", "Onion", "Garlic"],
      "Chicken Stir Fry": ["Chicken", "Broccoli", "Peppers", "Soy Sauce", "Ginger", "Rice"],
      "Chicken Biryani": ["Chicken", "Onion", "Garlic", "Rice", "Yogurt", "Spices"],
      "Salad": ["Mixed Greens", "Tomatoes", "Cucumbers", "Carrots", "Dressing"]
  }

  menu = list(ingredients.keys())
  ```

2. Let's say we get between 1 to a hundered customers each day and each of them orders a random dish from the menu. We'll store it in a list `ordered_dishes`.
  ```python
  import random
  
  no_of_customers = random.randint(1, 100)

  ordered_dishes = []
  for _ in range(no_of_customers):
      ordered_dishes.append(random.choice(menu))
  ```

3. We get a maximum of 100 customers per day so it's fine to store all the dishes in a list but what if we were to get a million customers per day. We don't want to have to store so much data in-memory till we process each dish. To fix it let's convert this to a generator expression to process one ordered dish at a time.
  ```python
  ordered_dishes = (random.choice(menu) for _ in range(no_of_customers))
  ```

4. To present the final dish, we need to first get the ingredients, cook them and serve the dish. All these functions don't really need to be generators but how else are we going to learn about generators.
  ```python
  def get_ingredients(dish):
    print(f"Get ingredients for {dish}...")
    yield from (ingredient for ingredient in ingredients[dish])
  
  def cook(dish_ingredients):
    for ing in dish_ingredients:
      print(f"Cooking ingredient {ing}...")
      yield f"Cooked {ing}"
  
  def serve(cooked_items):
    for item in cooked_items:
      print(f"Seasoning {item}...")
      yield f"Served {item} with a pinch of magic"
  ```

5. Now creating the pipeline and capturing the output.
  ```python
  chef_pipeline = ((dish, serve(cook(get_ingredients(dish)))) for dish in ordered_dishes)
  for final_dish in chef_pipeline:
      for item in final_dish[1]:
          print(item)
      print("Served", final_dish[0], "\n")
  ```

6. Final script
  ```python
  import random
  
  ingredients = {
      "Pizza": ["Flour", "Yeast", "Tomato Sauce", "Mozzarella Cheese", "Pepperoni"],
      "Spaghetti": ["Pasta", "Ground Beef", "Tomato Sauce", "Onion", "Garlic"],
      "Chicken Stir Fry": ["Chicken", "Broccoli", "Peppers", "Soy Sauce", "Ginger", "Rice"],
      "Chicken Biryani": ["Chicken", "Onion", "Garlic", "Rice", "Yogurt", "Spices"],
      "Salad": ["Mixed Greens", "Tomatoes", "Cucumbers", "Carrots", "Dressing"]
  }
  
  menu = list(ingredients.keys())
  
  no_of_customers = random.randint(1, 100)
  
  ordered_dishes = (random.choice(menu) for _ in range(no_of_customers))
  
  def get_ingredients(dish):
    print(f"Get ingredients for {dish}...")
    yield from (ingredient for ingredient in ingredients[dish])
  
  def cook(dish_ingredients):
    for ing in dish_ingredients:
      print(f"Cooking ingredient {ing}...")
      yield f"Cooked {ing}"
  
  def serve(cooked_items):
    for item in cooked_items:
      print(f"Seasoning {item}...")
      yield f"Served {item} with a pinch of magic"
  
  chef_pipeline = ((dish, serve(cook(get_ingredients(dish)))) for dish in ordered_dishes)
  for final_dish in chef_pipeline:
      for item in final_dish[1]:
          print(item)
      print("Served", final_dish[0], "\n")
  ```

This is very practical for a one man operation but what if it were a restraunt, multiple waiters, mutiple chefs, serving multiple. In that case we'll have to dive into concurrency, that's a story for another day. Till then check out these links for more on generators:
- https://peps.python.org/pep-0255/
- https://linuxgazette.net/100/pramode.html

[generators]: https://wiki.python.org/moin/Generators

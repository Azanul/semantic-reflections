---
layout: post
title:  "Cooking parallely in Python"
date:   2024-04-16 17:05:00 +0530
categories: python expert
---

Parallelism refers to multiple cores or processors to achieve genuine speedup which is very helpful for CPU-based tasks.

There are multiple ways one can achieve parallelism in Python:
1. Multiprocessing
2. Parallel processing libraries (e.g., Joblib, Dask, Ray, etc.)

Let's continue running a restraunt, we're reusing the same example used in last article.

## 1. Multiprocessing

Multiprocessing is a parallel processing technique that uses multiple processes to execute tasks. Each process has its own memory space, allowing true parallel execution of tasks on multiple CPU cores.
We avoid having to worry about the GIL since we're using subprocesses (which have their own memory space) instead of threads (which share a memory space and GIL keeps multiple threads from accessing the same memory at the same time).

1. Let's create a `Pool` with 3 processes and map the ordered dishes with our `prepare_dish` function.
    ```python
    from multiprocessing import Pool
    
    with Pool(processes=3) as pool:
        served_dishes = pool.map(prepare_dish, ordered_dishes())
    ```

2. Full script
    ```python
    from multiprocessing import Pool
    import random
        
    ingredients = {
        "Pizza": ["Flour", "Yeast", "Tomato Sauce", "Mozzarella Cheese", "Pepperoni"],
        "Spaghetti": ["Pasta", "Ground Beef", "Tomato Sauce", "Onion", "Garlic"],
        "Chicken Stir Fry": ["Chicken", "Broccoli", "Peppers", "Soy Sauce", "Ginger", "Rice"],
        "Chicken Biryani": ["Chicken", "Onion", "Garlic", "Rice", "Yogurt", "Spices"],
        "Salad": ["Mixed Greens", "Tomatoes", "Cucumbers", "Carrots", "Dressing"]
    }
    
    menu = list(ingredients.keys())
    
    no_of_customers = random.randint(1, 10)
    
    def ordered_dishes():
        for _ in range(no_of_customers):
            order = random.choice(menu)
            print(f"I- Ordered {order}")
            yield order
    
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
        
    def prepare_dish(dish):
        for op in serve(cook(get_ingredients(dish))):
            print(op)
        print(f"O- Served {dish}")
        return f"Served {dish}"
    
    with Pool(processes=3) as pool:
        served_dishes = pool.map(prepare_dish, ordered_dishes())
    
    for dish in served_dishes:
        print(dish)
    ```

## 2. Parallel processing libraries

Python offers various libraries like Joblib, Dask and Ray that simplify parallelism and can handle tasks across multiple cores efficiently (even multiple different machines with Dask & Ray).
I'm sure you can find more tools and libraries which aim to attain the same with their own pros and cons. We're going to check out how these three work and how to use them.
In this article we'll only checkout Joblib and cover Dask & Ray in a future article where we'd prepare ingredients in a different restraunt and cook them in a different restraunt.

1. The function `Parallel` here is pretty self explainatory. The function `delayed` is meant to avoid calling the fucntion immediately, instead delay it and wait to be called elsewhere. It creates a reference to the function to be called along with args and kwargs.
    ```python
    from joblib import Parallel, delayed
    
    served_dishes = Parallel(n_jobs=3)(delayed(prepare_dish)(dish) for dish in ordered_dishes())
    ```

2. Full script
    ```python
    from joblib import Parallel, delayed
    import random
        
    ingredients = {
        "Pizza": ["Flour", "Yeast", "Tomato Sauce", "Mozzarella Cheese", "Pepperoni"],
        "Spaghetti": ["Pasta", "Ground Beef", "Tomato Sauce", "Onion", "Garlic"],
        "Chicken Stir Fry": ["Chicken", "Broccoli", "Peppers", "Soy Sauce", "Ginger", "Rice"],
        "Chicken Biryani": ["Chicken", "Onion", "Garlic", "Rice", "Yogurt", "Spices"],
        "Salad": ["Mixed Greens", "Tomatoes", "Cucumbers", "Carrots", "Dressing"]
    }
    
    menu = list(ingredients.keys())
    
    no_of_customers = random.randint(1, 10)
    
    def ordered_dishes():
        for _ in range(no_of_customers):
            order = random.choice(menu)
            print(f"I- Ordered {order}")
            yield order
    
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
        
    def prepare_dish(dish):
        for op in serve(cook(get_ingredients(dish))):
            print(op)
        print(f"O- Served {dish}")
        return f"Served {dish}"
    
    served_dishes = Parallel(n_jobs=3)(delayed(prepare_dish)(dish) for dish in ordered_dishes())
    
    for dish in served_dishes:
        print(dish)
    ```

## Pros & Cons

| Feature	| Multiprocessing	| Parallel Processing Libraries (Joblib) |
| ------------|----------|--------------------------- |
| Concurrency Model |	Multi-process |	Multi-threaded/Process, based on selected backend (loky or threading) |
| I/O-bound tasks |	Overhead for process creation |	Less overhead |
| Memory Usage |	High (due to multiple processes) |	Moderate (less memory overhead compared to multiprocessing) |
| Complexity |	Slightly complex (process management) |	Simpler (higher-level APIs, easier to use than multiprocessing) |

## Moving forward

When choosing between multiprocessing and parallel processing libraries, consider the nature of your tasks. For CPU-bound tasks, both methods offer good performance. However, for I/O-bound tasks, parallel processing libraries may provide a more efficient solution due to lower overhead.
Remember to always test and benchmark your code to determine which method and which specific library best suits your specific requirements.
If you're dealing with big data loads or anything that requires high computational capacity then single machine parallelism is not going to solve your issue. For this we'll look into distributed processing/computing in python.
Till then, here are a few good resources to explore:


- https://docs.python.org/3/library/multiprocessing.html
- https://joblib.readthedocs.io/

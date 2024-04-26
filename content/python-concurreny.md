+++
title = "Cooking concurrently in Python"
date = 2024-04-12
[taxonomies]
tags = ["python", "advanced"]
+++

Concurrency refers to the ability of a program to manage multiple tasks seemingly at the same time. While a single CPU core can only execute one instruction at a time, concurrency allows programs to juggle multiple tasks by rapidly switching between them. This creates the illusion of parallelism, enhancing responsiveness and improving performance for I/O bound workloads.

There are two ways one can achieve concurrency in Python:
1. Multithreading
2. Asynchronous Programming (Asyncio)

Let's explore them one by one. This time we’re running a restraunt, we're extending the example used in article. We're going to have 3 chefs this time.

## 1. Multithreading

Threads are lightweight units of execution within a single process. They share the same memory space and resources but execute instructions independently.
Multiple threads can be running concurrently, but due to the Global Interpreter Lock (GIL) in Python's traditional CPython implementation, only one thread can execute Python bytecode at a time. There are other Python implementations that one can choose from, depending on the use case.

1. Let's redefine `ordered_dishes` to also print a message when an order is made.
    ```python
    def ordered_dishes():
        for _ in range(no_of_customers):
            order = random.choice(menu)
            print(f"I- Ordered {order}")
            yield order
    ```

2. Let's define a function `prepare_dish` wrapping our food pipeline logic.
    ```python
    def prepare_dish(dish):
        for op in serve(cook(get_ingredients(dish))):
            print(item)
        print(f"O- Served {dish}")
        return f"Served {dish}"
    ```

3. Let's create a thread pool with 3 workers and map `prepare_dish` with the dishes being ordered.
    ```python
    from concurrent.futures import ThreadPoolExecutor

    with ThreadPoolExecutor(max_workers=3) as executor:
        future_dishes = executor.map(prepare_dish, ordered_dishes())

    for dish in future_dishes:
        print(dish)
    ```

4. Full script
    ```python
    from concurrent.futures import ThreadPoolExecutor
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
    
    with ThreadPoolExecutor(max_workers=3) as executor:
        future_dishes = executor.map(prepare_dish, ordered_dishes())
    
    for dish in future_dishes:
        print(dish)
    ```

## 2. Asynchronous Programming (Asyncio)

Asynchronous programming is a programming paradigm that allows for non-blocking operations. Instead of waiting for a task to complete before moving on to the next one, asynchronous programming enables tasks to run in the background, freeing up resources and potentially improving performance.
Python's `asyncio` module provides a way to write concurrent code using the `async` and `await` syntax. It is particularly useful for I/O-bound and high-level structured network code.

1. Let's make `prepare_dish` an asynchronous function by prefixing the definition with `async` keyword.
    ```python
    async def prepare_dish(dish):
        ...
    ```

2. Let's create a pool with 3 workers and call `prepare_dish` with the dishes being ordered, similar to what we did with threads. To attain this, we're going to use a `Semaphore`. `async` and `await` can only be used inside an async function.
    ```python
    semaphore = asyncio.Semaphore(3)

    async def chef_task(dish):
        async with semaphore:
            return await prepare_dish(dish)
    ```

3. Wrap it all in an async `main` function and run it with `asyncio.run`. We use `asyncio.create_task` to run tasks concurrently. `asyncio.gather` aggregates the results of every coroutine passed to it.
    ```python
    async def main():
        future_dishes = []
        for dish in ordered_dishes():
            task = asyncio.create_task(chef_task(dish))
            future_dishes.append(task)
    
        served_dishes = await asyncio.gather(*future_dishes)
    
        for dish in served_dishes:
            print(dish)
    
    
    asyncio.run(main())
    ```

4. Full script
    ```python
    import asyncio
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
        
    async def prepare_dish(dish):
        for op in serve(cook(get_ingredients(dish))):
            print(op)
        print(f"O- Served {dish}")
        return f"Served {dish}"
    
    semaphore = asyncio.Semaphore(3)
    
    async def chef_task(dish):
        async with semaphore:
            return await prepare_dish(dish)
    
    async def main():
        future_dishes = []
        for dish in ordered_dishes():
            task = asyncio.create_task(chef_task(dish))
            future_dishes.append(task)
    
        served_dishes = await asyncio.gather(*future_dishes)
    
        for dish in served_dishes:
            print(dish)
    
    
    asyncio.run(main())
    ```

## Pros & Cons

| Feature	| Asyncio	| Multithreading (CPython) |
| ------------|----------|--------------------------- |
| Concurrency Model |	Single-threaded, event loop driven |	Multi-threaded |
| CPU-bound tasks	| Not ideal (overhead for context switching) |	Limited benefit due to GIL (serializes execution) |
| I/O-bound tasks |	Well-suited (handles waiting efficiently) |	Less efficient (overhead for thread management) |
| Memory Usage |	Lower (fewer threads) |	Higher (more threads) |
| Complexity |	Simpler (fewer potential race conditions) |	More complex (requires synchronization) |
| Error Handling |	Easier to reason about errors	| Error handling can be trickier (race conditions) |
| Learning Curve |	Steeper (different programming paradigm) |	Easier to learn (familiar thread concepts) |

## Moving forward

If you're not using CPython (e.g., Jython, IronPython), multithreading can potentially utilize multiple cores for CPU-bound tasks more effectively, depending on the implementation.
For highly CPU-bound tasks in CPython, consider libraries like multiprocessing which can leverage multiple cores by spawning separate processes instead of threads (processes don't share the GIL). We'll look deeper into multiprocessing in a different post.
Till then here are a few good resources to go through:
- [https://docs.python.org/3/library/asyncio-task.html](https://docs.python.org/3/library/asyncio-task.html)
- [https://docs.python.org/3/library/threading.html](https://docs.python.org/3/library/threading.html)


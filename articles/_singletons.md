# Singletons: Objects that can be instantiated only once

When developing larger programs, being aware of different patterns can greatly help us solve problems even before they arise. One of those patterns is the creation of singletons, which are nothing else but objects that can be instantiated only once. In Python, we are exposed to singletons since the beginning, even if we are not aware of them. In this article we are going to discuss how singletons permeate our every day programming and how we can bring them a step further. 

## Every Day Singletons
Before we dive into singletons, it is important to remember how Python behaves when it comes to mutable and immutable data types. A list, for example, is mutable, which means we can change its contents without creating a new object. For example:

```pycon
>>> var1 = [1, 2, 3]
>>> var2 = var1
>>> var1[0] = 0
>>> print(var2)
[0, 2, 3]
```

!!! note
    If the output of the code above puzzles you, I strongly advise you to check out the article on mutable and immutable data types. 

When we have two lists like ``var1`` and ``var2``, we can see whether they have the same content:

```pycon
>>> var1 == var2 
True
```

But we can also see if they are actually the same object:

```pycon
>>> var1 is var2
True
```

However, we can also do the following: 

```pycon
>>> var1 = [1, 2, 3]
>>> var2 = [1, 2, 3]
>>> var1 == var2
True
>>> var1 is var2
False
```

In this case, both ``var1`` and ``var2`` have the same values ``[1, 2, 3]`` but they are not the same object. That is why ``var1 is var2`` returns ``False``. 

However, Python programmers are likely exposed to the following syntax very early on:

```python
if var is None:
    print('Var is none')
```

At first sight the question that could pop anyone's mind is why can we use ``ìs`` in the example above. And the answer is that ``None`` is a special type of object, one that can be instantiated only once. Let's see some examples:

```pycon
>>> var1 = None
>>> var2 = None
>>> var1 == var2
True
>>> var1 is var2
True
>>> var3 = var2
>>> var3 is var1
True
>>> var3 is None
True
```

This means that through our code, there can be only one ``None``, and any variable that references it, will actually reference the exact same object, unlike what happened when we created two lists that had the same values. Together with ``None``, the other two common singletons are ``True`` and ``False``:

```pycon
>>> [1, 2, 3] is [1, 2, 3]
False
>>> None is None
True
>>> False is False
True
>>> True is true
True
```

That completes the triad of singletons that Python programmers encounter daily. It explains why we can use the ``is`` syntax when checking if a variable is ``None``, ``True`` or ``False``. This is only the beginning regarding singletons.

## Small integer singletons
Python defines other singletons that are not that obvious, and are mostly there because of memory and speed efficiency. Such is the case of small integers in the rage -5 to 256.

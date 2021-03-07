# Slots

## Dynamic attribute creation
One of the advantages of objects in Python is that they can store any number of attributes. Moreover, these attributes can be created dynamically, when the program runs, and not only when the class is defined. Let's start with a simple example to get the gist of it:

```python
class Person:
    def __init__(self, name):
        self.name = name
```

The class ``Person`` can be used like this:

```pycon
>>> me = Person('Aquiles')
>>> print(me.name)
'Aquiles'
```

The class ``Person`` creates an attribute ``name`` when we instantiate it, and we assign any value we want. In the example above, we used ``'Aquiles'``. We print the attribute ``name`` of the object ``me`` to show that it is behaving as expected. Python objects allow us to modify the attribute or to create new attributes:

```pycon
>>> me.name = 'John'
>>> me.last_name = 'Doe'
>>> print(me.name)
'John'
>>> print(me.last_name)
'Doe'
```

First, we replace the value of ``name`` at runtime. Later, we create a new attribute ``last_name`` that is not part of the original design of ``Person``. This pattern can be both extremely useful and dangerous. Imagine we have a complex object, for example to control a camera. We define an attribute ``exposure_time`` when we develop the ``Camera`` class. If later on we make a mistake and use ``exp_time`` instead, there will be no warning, and the error can go unnoticed until it is too late:

```pycon
>>> camera = Camera()
>>> camera.exposure_time = '5ms'
>>> camera.exp_time = '10ms'
```

## Quick intro to slots
Limiting the creation of attributes at runtime can be a great advantage and Python offers a very simple syntax to achieving it:

```python
class Person:
    __slots__ = 'name'
    def __init__(self, name):
        self.name = name
```

And this time, if we run the same coda we ran earlier, we will get a different outcome:

```pycon
>>> me = Person('Aquiles')  
>>> print(me.name)  
'Aquiles'
>>> me.name = 'John'  
>>> print(me.name)  
'John'
>>> me.last_name = 'Doe'
Traceback (most recent call last):
  File "/Users/aquiles/Documents/Web/pftl_code/code/_slots/aa.py", line 11, in <module>
    me.last_name = 'Doe'
AttributeError: 'Person' object has no attribute 'last_name'
```

We could alter the ``name`` attribute at runtime, but ``last_name`` gave and ``AttributeError``. If we modify the slots to include ``last_name``, then the output would be different:

```python
class Person:
    __slots__ = 'name', 'last_name'
    def __init__(self, name):
        self.name = name
```

And this time there will be no ``AttributeError``:

```pycon
>>> me = Person('Aquiles')
>>>me.last_name = 'Doe'
>>> print(me.last_name)
'Doe'
```

In the examples above it is clear that by using slots, we can limit the dynamic creation of attributes. It is important to note, however, that classes tend to get much more sophisticated and store many different attributes. Keeping ``__slots__`` up to date can quickly become a hassle. 

There are two things that are worth noting. One is that class attributes (i.e. attributes defined at a class level) automatically become read-only:

```python
class Person:
    __slots__ = 'name', 'last_name'
    age = 35

    def __init__(self, name):
        self.name = name
```

And if we try to change the age after instantiating the class we get the following error:

```pycon
>>> me = Person('Aquiles')
>>> print(me.age)
35
>>> me.age = 50
Traceback (most recent call last):
  File "/Users/aquiles/Documents/Web/pftl_code/code/_slots/aa.py", line 16, in <module>
    me.age = 50
AttributeError: 'Person' object attribute 'age' is read-only
```

We'll dive more into the reasons behind this behavior in the following section. 

## Where are attributes stored, the __dict__

## Slots and inheritance

## Impact of slots on size and speed

## Alternatives to using slots

## Conclusions: Deciding when is worth using slots

# Slots
When we create classes, one of the biggest challenges is understanding how to handle dynamic attribute creation. Slots have the benefit of limiting attribute creation at runtime. In this article, we will explore how slots work, including a quick overview of how classes store attributes internally. 

## Dynamic attribute creation
One of the advantages of objects in Python is that they can store any number of attributes. Moreover, these attributes can be created dynamically when the program runs and not only when the class is defined. Let's start with a simple example to get the gist of it:

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

First, we replace the value of ``name`` at runtime. Later, we create a new attribute, ``last_name`` that is not part of the original design of ``Person``. This pattern can be both handy and dangerous. Imagine we have a complex object, for example, to control a camera. We define an attribute ``exposure_time`` when we develop the ``Camera`` class. If later on we make a mistake and use ``exp_time`` instead, there will be no warning, and the error can go unnoticed until it is too late:

```pycon
>>> camera = Camera()
>>> camera.exposure_time = '5ms'
>>> camera.exp_time = '10ms'
```

## Quick intro to slots
Limiting the creation of attributes at runtime can be a great advantage, and Python offers a straightforward syntax to achieving it:

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

In the examples above, it is clear that we can limit the dynamic creation of attributes by using slots. However, it is essential to note that classes tend to get much more sophisticated and store many different attributes. Keeping ``__slots__`` up to date can quickly become a hassle. 

Two things are worth noting. One is that class attributes (i.e., attributes defined at a class level) automatically become read-only:

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
Objects must remember the attributes to which they have access to retrieve them or give an error if they can't be found. In Python, this is done through a dictionary called ``__dict__``. We can see how it works with the same example we used earlier:

```python
class Person:
    birth_year = 1986
    def __init__(self, age, name):
        self.age = age
        self.name = name

    def print_name(self):
        print(self.name)
```

We can inspect the dictionary of both the Person class and of one of its instances:

```pycon
>>> print(Person.__dict__)
{'__module__': '__main__', 'birth_year': 1986, '__init__': <function Person.__init__ at 0x7fd890028ee0>, 'print_name': <function Person.print_name at 0x7fd89046ca60>, '__dict__': <attribute '__dict__' of 'Person' objects>, '__weakref__': <attribute '__weakref__' of 'Person' objects>, '__doc__': None}
>>> me = Person(35, 'Aquiles')
>>> print(me.__dict__)
{'age': 35, 'name': 'Aquiles'}
```

Note, first, that we are not using slots; it is a plain class. The dictionary of the class stores references to its attributes ``birth_year`` and the methods ``__init__`` and ``print_name``. It has no information about ``age`` or ``name`` because these attributes get created only *after* the class is instantiated. 

On the other hand, the dictionary of the object ``me`` has both ``name`` and ``age`` but nothing else. If we would like to access the value stored in the dictionary, we can do the following:

```python
me.__dict__['name']
```

If we add slots, however, things will change. Let's what happens to the dictionary in the following example:

```python
class Person:
    __slots__ = 'age', 'name'
    birth_year = 1986

    def __init__(self, age, name):
        self.age = age
        self.name = name

    def print_name(self):
        print(self.name)
```

We kept the ``birth_year`` class attribute, but we added the slots for ``age`` and ``name``. If we repeat what we did earlier:

```pycon
>>> print(Person.__dict__)
{'__module__': '__main__', '__slots__': ('age', 'name'), 'birth_year': 1986, '__init__': <function Person.__init__ at 0x7ffdf0118ee0>, 'print_name': <function Person.print_name at 0x7ffde032ba60>, 'age': <member 'age' of 'Person' objects>, 'name': <member 'name' of 'Person' objects>, '__doc__': None}
>>> me = Person(35, 'Aquiles')
>>> print(me.__dict__)
Traceback (most recent call last):
[...]
AttributeError: 'Person' object has no attribute '__dict__'
```

The only difference between the earlier code and this one is that we added the ``__slots__`` class attribute. We can notice that the class dictionary is different. Now there are both ``age': <member 'age' of 'Person' objects>`` and ``'name': <member 'name' of 'Person' objects>``. Moreover, if we explore the object ``me``, we see that it has no  ``__dict__``. It is fair, therefore, to ask where are these attributes stored, and the answer is close by:

```pycon
>>> print(me.__slots__)
('age', 'name')
```

To wrap up what we have just seen, when we use slots, we create objects with no ``__dict__`` associated with them, but rather a ``__slots__`` tuple. It is a significant distinction since tuples are immutable in Python. Moreover, the attributes that we defined in the slots show directly on the class dictionary, and this was not the case before. 

We can explore a bit more this additions by looking at what their types are. For example, we can do the following:

```pycon
>>> print(type(Person.__dict__['birth_year']))
<class 'int'>
>>> print(type(Person.__dict__['age']))
<class 'member_descriptor'>
```

And this is the quid of the question regarding slots. When we define them in a class, they are automatically created as descriptors. We have written a [complete article about descriptors](https://www.pythonforthelab.com/blog/data-descriptors-bringing-attributes-next-level/) that may be worth checking out to go deeper with the understanding. It is the reason why we get the read-only attributes if we don't specify them in the slots. 

### Adding the dictionary to the slots
We saw that when we define slots for a class, its objects do not have a ``__dict__`` to store their attributes. Therefore, we could ask ourselves what would happen if we add the dictionary to the list of slots. Let's try it out:

```python
class Person:
__slots__ = 'age', 'name', '__dict__'

def __init__(self, age, name):
    self.age = age
    self.name = name
    self.birth_year = 1986
```

And we can now use it as we have always done:
```pycon
>>> me = Person(35, 'Aquiles')
>>> print(me.__dict__)
{'birth_year': 1986}
>>> me.new_var = 10
>>> print(me.__dict__)
{'birth_year': 1986, 'new_var': 10}
```

We have created a hybrid class in which some attributes are slots, and therefore they do not appear in the dictionary. However, since the class itself has a dictionary, we can dynamically create new attributes. Whether this pattern is useful (or even correct) can be subject to discussion. We are not going to engage with it right now. 

## Slots and inheritance
The final important topic to cover about slots is how they behave with inheritance. Let's start creating a new class that inherits from ``Person`` and see what happens:

```python
class Person:
__slots__ = 'age', 'name'
def __init__(self, age, name):
    self.age = age
    self.name = name

class Student(Person):
    def __init__(self, age, name, course):
        super(Student, self).__init__(age, name)
        self.course = course
```

In the example above, ``Studen`` inherits from ``Person`` but it does not define new slots. Therefore, it will behave like any other class:

```pycon
>>> me = Student(35, 'Aquiles', 'Physics')    
>>> print(me.__dict__)
{'course': 'Physics'}
>>> me.new_var = 10
>>> print(me.__dict__)
{'course': 'Physics', 'new_var': 10}
>>> print(me.__slots__)
('age', 'name')
```

``Student`` handles dynamic attribute creation, and we still have access to the slots defined in the parent class ``age`` and ``name``. This means that if we inherit from a class, we should not worry whether it defined slots or not. The child class will behave precisely as we design it to behave. 

The other possibility is to define slots in the child class but not in the parent class, like this:

```python
class Person:
def __init__(self, age, name):
    self.age = age
    self.name = name

class Student(Person):
    __slots__ = 'course'
    def __init__(self, age, name, course):
        super(Student, self).__init__(age, name)
        self.course = course
```

And, surprisingly, the code above still works:

```pycon
>>> me = Student(35, 'Aquiles', 'Physics')
>>> print(me.__dict__)
{'age': 35, 'name': 'Aquiles'}
>>> me.new_var = 10
>>> print(me.__dict__)
{'age': 35, 'name': 'Aquiles', 'new_var': 10}
>>> print(me.__slots__)
course
```

Therefore,  if either the parent or the child defines the ``__dict__``, then the objects will also have a dict and will be able to accept dynamically created attributes. The only way in which can leverage slots is if both parent and child define them:

```python
class Person:
    __slots__ = 'age', 'name'
    def __init__(self, age, name):
        self.age = age
        self.name = name

class Student(Person):
    __slots__ = 'course'
    def __init__(self, age, name, course):
        super(Student, self).__init__(age, name)
        self.course = course
```

And then we'll see the behavior that would be expected from having slots:

```pycon
>>> me = Student(35, 'Aquiles', 'Physics')
>>> print(me.__slots__)
course
>>> me.new_var = 10
Traceback (most recent call last):
[...]
AttributeError: 'Student' object has no attribute 'new_var'
```

## Impact of slots on size and speed of code
One of the reasons behind adding slots to Python was to have faster attribute access. This was discussed in [this blog post](https://python-history.blogspot.com/2010/06/inside-story-on-new-style-classes.html) from 2010, summarizing all the improvements done to classes. Essentially, slots were added to overcome potential impacts in performance. By defining slots, the programs can have a faster lookup of the data stored. Whether an increase of lookup times of around 15% impacts the overall code will depend on how often we perform the task. 

The other impact of slots is lower memory usage. Since slots prevent creating a dictionary and a *weakref* (which didn't discuss in this article), each object created will require less memory. For code creating few hundreds of objects, the impact may be negligible. Still, if we are creating millions of objects, the effect can be tremendous. For example, [SQLAlchemy has a measurable effect on the use of slots](https://docs.sqlalchemy.org/en/14/changelog/migration_10.html#significant-improvements-in-structural-memory-use).

## Conclusions: Deciding when is worth using slots
Slots are a feature worth keeping in mind when developing code and, more importantly, when studying other's code. Using them for speed and memory improvements is likely to be far-fetched for most programs. It is essential to keep in mind that if a program was optimized using slots, we could easily ruin those optimizations by careless inheritance. 

On the other hand, limiting the attribute creation at runtime can have a broad set of application contexts, even if it was not the original motivation for implementing slots in Python. For example, when developing code to control hardware, such as a camera, users may make mistakes such as using ``exposure`` instead of ``exposure_time`` to change a parameter. Due to dynamic attribute creation, there won't be any error shown on the screen. 

Slots are not the only way of solving these issues, but they are straightforward to implement. 
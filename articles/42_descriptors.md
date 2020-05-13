# Data Descriptors: Bringing Attributes to the Next level

## Properties are Descriptors
Once you fully embrace the object oriented nature of Python, there is a high chance that you cross interesting tools, such as the @propery decorator. Let's quickly see an example of how it works and what it can do for us:

```python
class MyClass:
    _var = 0

    @property
    def var(self):
        print('Getting Var')
        return self._var
```

```pycon
>>> my_class = MyClass()
>>> print(my_class.var)
Getting Var
0
```

We can see that by using the ``@property`` decorator, we have changed the behavior of a method, ``var`` to look like an attribute of the class ``MyClass``. We wen't one step further, and we actually added some extra functionality: whenever we read the value of ``var``, a line will be printed to the screen informing us about it. This pattern have very interesting possibilities, that we will discuss in the rest of the article. 

As a side note, Python does not have *private* attributes. It means that whatever we define in a class can be access and modified also from outside of it. Using the underscore for ``_var`` is just a way of signaling other developers that they should not change ``_var`` directly, but that it should be changed only within the class itself. IDE's such as PyCharm will warn us if we ever try to alter one such variable, but there is nothing more that we can do. 

Before we move to more complicated topics, there is something worth mentioning. In the enthusiasm of learning a new topic sometiems important considerations are overlooked. Whenever we encounter code that looks like this:

```pycon
>>> my_class.var
```

We actually expect execution to be close to instantaneous. If, for example, the method ``var`` includes connecting to a remote server and fetching information, the behavior of the program would become tougher to anticipate. It is hard to draw a line between clarity and usefulness. Always keep in mind that there is nothing that descriptors can do that can't be achieved with plain methods. 

Getting a value is less than half what properties can do. The other half is setting a value. The syntax is  straightforward:

```python
class MyClass:
    _var = 0

    @property
    def var(self):
        print('Getting var')
        return self._var

    @var.setter
    def var(self, value):
        print('Setting var')
        self._var = value
```

```pycon
>>> my_class = MyClass()
>>> print(my_class.var)
Getting var
0
>>> my_class.var = 2
Setting var
>>> print(my_class.var)
Getting var
2
```

When we used the decorator ``@property``, it actually altered the method ``var`` in such a way that we could use itself as a decorator. A fair question at this stage is what do we gain by using properties instead of methods. In my opinion, at this stage, it is only a matter of simplifying how the code looks. Compare this code:

```pycon
>>> my_class.get_var()
0
>>> my_class.set_var(2)
```

to this one:

```pycon
>>> print(my_class.var)
0
>>> my_class.var = 2
```

We could argue that the second is easier to read. Properties are very handy when we have a lot of methods for getting and setting values, because they can simplify the interface quite a lot. This happens when we control devices, for example, and each setting has a different method for reading and changing its value. 

Just to have an example of going one step further with properties and their behavior, we can use them to verify that the value we assign to ``var`` is an integer:

```python
@var.setter
def var(self, value):
    print('Setting var')
    if not isinstance(value, int):
        raise TypeError('Value must be integer')
    self._var = value
```

And we can test it:

```pycon
>>> my_class.var = 2.0
[...]
TypeError: Value must be integer
```

## Develop your own Descriptors
Using ``@property`` can extend our options when planning classes in Python, but at some point we need to go beyond the built-in tools. When we defined descriptors, we said that they are objects that define how they are ``get`` and ``set`` from the object that contains them. However, ``var`` does not implement anything like that. The real descriptor is ``@property``. So, let's start by developing our own object that can work in a similar way. 

```python
class MyDescriptor:
    _val = 0

    def __get__(self, instance, owner):
        print('Getting Descriptor')
        return self._val

    def __set__(self, instance, value):
        print('Setting Value')
        self._val = value


class MyClass:
    var = MyDescriptor()
```

``MyDescriptor`` is a class that defines two methods: ``__get__`` and ``__set__``. Perhaps you can already notice that the methods are very similar to how we defined ``var`` in the previous section. We can use the class like this:

```pycon
>>> my_class = MyClass()
>>> my_class.var
Getting Descriptor
0
>>> my_class.var = 2
Setting Value
>>> my_class.var
Getting Descriptor
2
```

We are not limited to having only one ``MyDescriptor`` in the class, we can define as many as we need:

```python
class MyClass:
    var = MyDescriptor()
    var1 = MyDescriptor()
    var2 = MyDescriptor()
```

In this way we can define as many attributes as we want, all with the same behavior. Every time we read them, a message will be printed, every time we set them, another message will be printed. However useful, this is also pushing us away from how properties work, because we are storing ``_var`` directly in the descriptor and not in the class itself. 

What we would like to achieve is the following behavior:

```python
class MyClass:
    _var = 0

    @MyDescriptor
    def var(self):
        print('Getting var')
        return self._var
```

We need to change ``MyDescriptor`` so we can use it as a decorator. We should also change its ``__get__`` method, so it uses the method defined in the class and not just a standard method defined in the descriptor. 

```python
class MyDescriptor:
    def __init__(self, fget):
        print('Instantiating descriptor')
        self.fget = fget

    def __get__(self, instance, owner):
        print('Getting Descriptor')
        return self.fget(instance)
```

Notice that we have included an ``__init__`` method that takes one argument: fget. When we develop decorators, the method being decorated will be the first argument. You can check [our article on decorators](https://www.pythonforthelab.com/blog/how-to-use-decorators-part-2/) to learn more. We also had to adapt the ``__get__`` method to use the ``fget`` function. Notice that the first argument of ``fget`` should be ``self``, and that is why we pass ``instance`` as the first argument. 

When we use it, we can have a clear look at the order in which things happen:

```pycon
>>> from descriptors import *
Instantiating descriptor
>>> my_class = MyClass()
print(my_class.var)
>>> my_class = MyClass()
>>> my_class.var
Getting Descriptor
Getting var
0
```

The instantiation of the descriptor happens at import time. Whenever the python interpreter goes through those lines of code, even if we never instantiate the class, the Descriptor will be instantiated. The rest of the code proceeds as usual, but not that first we get the descriptor, then we run the method for returning ``var``. 

As we saw earlier, this is only half the problem. Once we know how to get a value, we should aso see how to set it. Before we do anything, we can just try:

```pycon
>>> my_class = MyClass()
>>> print(my_class.var)
Getting Descriptor
0
>>> my_class.var = 2
>>> print(my_class.var)
2
```
The first time we access ``var``, we get the two lines printed on screen telling us that both the method and the descriptor worked as expected. However, after we set ``var = 2``, they stop working. This is very interesting, because what happened is that we overwrote the ``var``, which is a descriptor, by a plain integer. We need to prevent this, we can add an explicit ``__set__`` method that raises an exception:

```python
def __set__(self, instance, value):
    raise Exception('This is a read-only descriptor')
```

And with this the problem of overwriting ``var`` goes away. But, we are still far from done. What happens if we actually want to define a setter? Things slowly will get more interesting. We would like to use the following syntax:

```python
class MyClass:
    _var = 0

    @MyDescriptor
    def var(self):
        return self._var

    @var.setter
    def var(self, value):
        self._var = value
```

Right now, ``var`` is passed as the first argument of ``MyDescriptor``. The only way in which ``setter`` can exist is if it is a method of ``MyDescriptor``. Moreover, we should also store not only ``fget`` but also ``fset`` as attributes. We can start by improving the ``__init__`` and ``__set__`` methods:

```python
class MyDescriptor:
    def __init__(self, fget=None, fset=None):
        print('Instantiating descriptor')
        self.fget = fget
        self.fset = fset

    def __set__(self, instance, value):
        print('Setting Descriptor')
        self.fset(instance, value)
```

To quickly use the solution as is, we could define ``var`` with a more explicit syntax:

```python
class MyClass:
    _var = 0

    def get_var(self):
        return self._var

    def set_var(self, value):
        self._var = value

    var = MyDescriptor(get_var, set_var)
```

You can go ahead and test it to see that it works, even if it is not what we were after. By the way, the same syntax also works for ``property``. The ``setter`` method looks a bit more magical, but if we break it in parts we can understand it:

```python
    def setter(self, fset):
        return type(self)(self.fget, fset)
```

``setter`` takes one argument, ``fset`` which will be the decorated method. Now, we need to instantiate again the ``MyDecorator``, using the ``fget`` that we already know and passing the ``fset`` which is new. This is where the magic of ``type(self)`` comes into place. It returns the class to which the instance (``self``), belongs, and we can use it in the same way as we would use the ``MyDecorator``. 

This is the only missing part, now we can use the full syntax:

```python
class MyClass:
    _var = 0
    
    @MyDescriptor
    def var(self):
        return self._var

    @var.setter
    def var(self, value):
        self._var = value


my_class = MyClass()
print(my_class.var)
my_class.var = 2
print(my_class.var)
```

Of course there are a lot of different options that can be implemented. For example, we could raise an exception of ``fset`` does not exist, such as we did earlier. It would mean that ``var`` is "read-only". We could also define "set-only" properties, etc. But there is something important to discuss before going forward. 

We are using ``MyDescriptor`` as a decorator, and we managed to use ``setter`` without too much effort. Note that we used the name ``var`` for both the get and set methods. This is because Python binds those names to the class. If we use a different name for the setter, for example:

```python
class MyClass:
    _var = 0

    @MyDescriptor
    def var(self):
        return self._var

    @var.setter
    def var_setter(self, value):
        self._var = value
```

We would end up with two different attributes, ``var`` is read-only, and ``var_setter`` which can be read and set. However, the underlying value, ``_var`` is the same:

```python
my_class = MyClass()
print(my_class.var)
my_class.var_setter = 2
print(my_class.var_setter)
print(my_class.var)
```

The last two lines would print the same value to the screen. Therefore, we must be careful, because using different names is not wrong in itself, but it defeats the purpose of having a clear interface. Before we move on, there is something else that we can implement with what we already know. 

We could use ``MyDescriptor`` to specify some boundaries. If the value we try to set is beyond them, we raise an exception. We would like to be able to have a class that looks like this:

```python
class MyClass:
    _var = 0

    @MyDescriptor(val_min=0, val_max=3)
    def var(self):
        return self._var

    @var.setter
    def var(self, value):
        self._var = value
```

At this step, it is very important that you are familiar with decorators. If you are not, we recommend you to check [this article](https://www.pythonforthelab.com/blog/how-to-use-decorators-part-2/). Not that we instantiate ``MyDescriptor`` and then we use it as a decorator. The only way in which that can work is if we define the ``__call__`` method. We will also need to change the ``__init__`` in order to accomodate for the limits. The full class would look like this:

```python
class MyDescriptor:
    def __init__(self, fget=None, fset=None, val_min=None, val_max=None):
        print('Instantiating descriptor')
        self.val_min = val_min
        self.val_max = val_max
        self.fget = fget
        self.fset = fset

    def __call__(self, fget):
        return type(self)(fget, self.fset, self.val_min, self.val_max)

    def __get__(self, instance, owner):
        print('Getting Descriptor')
        return self.fget(instance)

    def __set__(self, instance, value):
        print('Setting Descriptor')
        if not self.val_min <= value <= self.val_max:
            raise ValueError(f'Value must be between {self.val_min} and {self.val_max}')
        self.fset(instance, value)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.val_min, self.val_max)
```

Most of the code is the same, but the flow is very different. The ``__call__`` method allows us to specify how a class can be used after it was instantiated. To understand it a bit better, let's simplify the code:

```python
class MyDescriptor:
    def __init__(self, val):
        self.val = val

    def __call__(self):
        print(self.val)
```

And we can use it like this:

```pycon
>>> my_descriptor = MyDescriptor(1)
>>> my_descriptor()
1
```

With the example above, we can see that ``__call__`` is the method that allows us to use ``MyDescriptor`` as a decorator, even after we have instantiated it. The pattern is the same we used for the ``setter`` method, we just assume that the argument that follows is the *getter* method. 

Back to the full example, the decorator will take care of checking if the value is between the specified range:

```pycon
>>> my_class = MyClass()
>>> my_class.var
Getting Descriptor
0
>>> my_class.var = 5
Setting Descriptor
[...]
ValueError: Value must be between 0 and 3
```

### Only Setters
In the examples above, both with properties and our custom descriptor, we have always assumed that the getter was going to be defined. However, it is also possible to have descriptors that can only be set, but not retrieved. It is hard to come up with situations in which this could be useful. With a property, it would look like this:

```python
class MyClass:
    _var = 0

    def var_setter(self, value):
        self._var = value

    var = property(None, var_setter)
```

And with our descriptor it would look like this:

```python
class MyClass:
    _var = 0

    var = MyDescriptor(val_min=0, val_max=3)

    @var.setter
    def var(self, value):
        self._var = value
```

You should study the code above to understand why we can use ``MyDescriptor`` both as a decorator and as a normal class. 

The sections above cover the classic approach to descriptors. However, since **Python 3.6**, objects include another method, ``__set_name__`` that allows to easily manipulate the owner of the descriptor. This behavior is where real new opportunities appear in a much more straightforward way of thinking. 

## Accessing the Owner Class with ``set_name``
Something that will happen at some point is that you would like to know where an attribute is defined. A possible pattern would be to register certain types of attributes on the class that holds them. For example, imagine you have a device in which you define settings and that you would like to know all the settings contained within that device. To achieve that, you should have attributes that are able to register themselves in the owner class. 

First, let's develop a solution to show what ``__set_name__`` can do, and then we can see the details. The descriptor is very similar to the one we developed earlier: 

```python
class MyDescriptor:
    def __init__(self, fget=None, fset=None):
        print('Instantiating descriptor')
        self.fget = fget
        self.fset = fset

    def __set_name__(self, owner, name):
        print(f'Setting name to {name}')
        if not hasattr(owner, '_descriptors'):
            setattr(owner, '_descriptors', [])

        owner._descriptors.append(name)

    def __get__(self, instance, owner):
        print('Getting Descriptor')
        return self.fget(instance)

    def __set__(self, instance, value):
        print('Setting Descriptor')
        self.fset(instance, value)

    def setter(self, fset):
        return type(self)(self.fget, fset)
```

The class to use this descriptor would be the same as before:

```python
class MyClass:
    _var = 0

    @MyDescriptor
    def var(self):
        return self._var

    @var.setter
    def var(self, value):
        self._var = value
```

And when we actually use it, we will see the behavior: 

```pycon
>>> from descriptors import MyClass
Instantiating descriptor
Instantiating descriptor
Setting name to var
>>> my_class = MyClass()
>>> print(my_class._descriptors)
['var']
```

Note that the name was assigned before we instantiated the class. This means that the ``__set_name__`` method had access to the owner class right at the moment of its definition. Finally, we see that we can store all the instances of ``MyDescriptor`` on a list. If we would have more than one, they would all appear there. 

We have chosen a list because it is the most straightforward application, but we could also use a dictionary, for example, to store not only the name but also the latest updated value, or the time of update, and we could use that as a cache for later retrievals. 

What we show here is one of the simplest patterns that can be achieved with ``__set_name__``, but also one of the most useful ones. Being able to register attributes on the owner class is of great help in many applications. 

### Taking care of inheritance
The method above has one big problem when dealing with inheritance. Let's quickly see what happens if we have a second class that inherits from ``MyClass``:

```python
class MyClass:
    _var = 0
    _var1 = 1

    @MyDescriptor
    def var(self):
        return self._var

    @var.setter
    def var(self, value):
        self._var = value

    @MyDescriptor
    def var1(self):
        return self._var1

class MyOtherClass(MyClass):
    _new_var = 2

    @MyDescriptor
    def new_var(self):
        return self._new_var
```

If we look at the list of descriptors, we will see that they are both the same:

```pycon
>>> my_class = MyClass()
>>> my_other_class = MyOtherClass()
>>> my_class._descriptors
['var', 'var1', 'new_var']
>>> my_other_class._descriptors
['var', 'var1', 'new_var']
```

This is a great time to check out what [mutable and immutable](https://www.pythonforthelab.com/blog/mutable-and-immutable-objects/) data types are in Python. What is happening is that the list of descriptors is mutable, and therefore it is shared. When the child appends a new value, it appears also at the parent class. 

Solving this problem is not trivial, but Hernán Grecco found a [very elegant solution](https://github.com/lantzproject/lantz-core/blob/b30a073296fb86fe652bc90893514e15ffbfe840/lantz/core/feat.py#L106) for Lantz, which I will explain here.
 
### Taking care of inheritance
#### Subclassing built-in data types 
First, we need to discuss something that may seem slightly esoteric for most, which is creating a child object from a standard data-type in Python. As a general rule, if you don't have a good reason to do it, it is probably a good idea to avoid this pattern. However, this time we do have a good reason. It works the same as always, let's assume we want to subclass a list, we do:

```python
class MyList(list):
    pass


var = MyList([1, 2, 3])
print(len(var))
```

``var`` has the same features a normal list has, so we can ``append``, iterate through the objects, etc. But we can now define new attributes to ``MyList``, as we would do with our objects:

```python
var.my_info = 'This is my info'
```

It would not have been possible to achieve the same with a plain list. When following this pattern, you should be extra careful not to overwrite methods from the list. For example, the following could lead to errors even though is not incorrect in itself:

```python
var.append = 'Update'
```

Before we continue, it is also important to refresh how we can set and get attributes to an object: 

```python
setattr(var, 'my_info', 'Updated Info')
getattr(var, 'my_info')
```

#### Checking ownership
The general idea to prevent children to propagate information to their parents would be to register the owner class in the list of descriptors itself. Every time a new descriptor is created, it checks whether the list of descriptors and the class that defines it belong to the same owner. If it is the same, then it just appends itself. If it is different, it means is a child creating a descriptor, and instead of appening, it creates a new list. 

We start by creating a custom list:

```python
class MyList(list):
    pass
```

And the secret now is to use this list to register the descriptors. We can add the following to ``MyDescriptor``:

```python
def __set_name__(self, owner, name):
    descriptors = getattr(owner, '_descriptors', None)
    if descriptors is not None:
        if getattr(descriptors, 'owner_class', None) != owner.__qualname__:
            owner._descriptors = MyList(descriptors)
            setattr(owner._descriptors, 'owner_class', owner.__qualname__)
        owner._descriptors.append(name)
    else:
        setattr(owner, '_descriptors', MyList())
        setattr(owner._descriptors, 'owner_class', owner.__qualname__)
        owner._descriptors.append(name)
```

Let's go line by line. First, we check if the owner has a ``_descriptors`` attribute. If it has one, it probably means it was created by another descriptor, but we should check whether it was another descriptor in the same class or in the parent class. We are going to use the attribute ``owner_class`` in the custom list. The dunder ``__qualname__`` returns the *qualified name*, which is the full path to the class (including the ``.``) and therefore should be unique through your program. 

If the registered qualified name is different from the owner's name, it means we are probably dealing with an inheritance. So, we we have to do is create a new ``_descriptors`` attribute. With this we unlink the parent from the child attribute. Right after creating the list, we set the attribute ``owner_class`` to the qualified name of the owner, so it can be used later on. Once we know we are dealing with the proper ``_descriptors``, we append the name of the current descriptor. 

The last block is taking care of the situation where the owner does not have a ``_descriptors`` attribute defined. Depending on what you are building, you may already know that the attribute has to be there. But it has no more complications, it just forces the owner to have a list, with the proper owner class, and then appends the name. 

With the same definition of the classes that we had before, we can check again if we are getting the proper list of descriptors:

```python
my_class = MyClass()
my_other_class = MyOtherClass()
print(my_class._descriptors)
# ['var', 'var1']
print(my_other_class._descriptors)
# ['var', 'var1', 'new_var']
```

So now you see that inheritance is working only from parent to children, as expected. 

## Conclusions
We have focused this article on using descriptors as decorators, similar to how @property works. We do believe this is one of the most common patterns for descriptors, but it is by no means the only one. First, you don't need to use them as decorators for methods, they could be attributes directly defined in classes. If you check the [official documentation](https://docs.python.org/3/howto/descriptor.html#static-methods-and-class-methods) you will see examples that drill deeper into manipulating the ``__dict__`` of the objects. This degree of complexity is, however, seldom required, but please let us know in the comments if it can be useful for you. 

The *descriptor protocol* is an incredibly useful tool when you need to manipulate the class where special attributes are defined. A very common situation is what we showed in the last section: registering certain attributes in a list. We could go one step further and define a cache, timeouts, and more. It is not something everyone will need every time, but when you wonder how to have access to the owner class when you are defining an attribute, descriptors are the solution. 
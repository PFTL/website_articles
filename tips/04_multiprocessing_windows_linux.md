# Differences of Multiprocessing on Windows and Linux

Multiprocessing is an excellent package if you ever want to speed up your code without leaving Python. When I started working with multiprocessing, I was unaware of the differences between Windows and Linux, which set me back several weeks of development time on a relatively big project. Let's quickly see how multiprocessing works and where Windows and Linux diverge. 

The quickest way of showing how to use multiprocessing is to run a simple function without blocking the main program:

```python
import multiprocessing as mp
from time import sleep


def simple_func():
    print('Starting simple func')
    sleep(1)
    print('Finishing simple func')


if __name__ == '__main__':
    p = mp.Process(target=simple_func)
    p.start()
    print('Waiting for simple func to end')
    p.join()
```

Which outputs the following:

```bash
Waiting for simple func to end
Starting simple func
Finishing simple func
```

The output is what we were expecting. Let's go to the core of the problem at hand by studying how this code behaves: 

```python
import multiprocessing as mp
from time import sleep


print('Before defining simple_func')

def simple_func():
    print('Starting simple func')
    sleep(1)
    print('Finishing simple func')


if __name__ == '__main__':
    p = mp.Process(target=simple_func)
    p.start()
    print('Waiting for simple func to end')
    p.join()
```

If we run this code on Windows, we get the following output:

```bash
Before defining simple_func
Waiting for simple func to end
Before defining simple_func
Starting simple func
Finishing simple func
```

While on Linux we get the following output:

```bash
Before defining simple_func
Waiting for simple func to end
Starting simple func
Finishing simple func
```

It does not look like much, except for the second ``Before defining simple_func``, and this difference is crucial. On **Linux**, when you start a child process, it is *Forked*. It means that the child process inherits the memory state of the parent process. On Windows (and by default on Mac), however, processes are *Spawned*. It means that a new interpreter starts and the code reruns. 

It explains why, if we run the code on Windows, we get twice the line ``Before defining simple_func``. As you may have noticed, this could have been much worse if we wouldn't include the ``if __main__`` at the end of the file, let's check it out. On Windows, it produces a very long error, that finishes with:

```bash
RuntimeError: 
        An attempt has been made to start a new process before the
        current process has finished its bootstrapping phase.

        This probably means that you are not using fork to start your
        child processes and you have forgotten to use the proper idiom
        in the main module:

            if __name__ == '__main__':
                freeze_support()
                ...

        The "freeze_support()" line can be omitted if the program
        is not going to be frozen to produce an executable.
```

While on Linux, it works just fine. It may not look like much, but imagine you have some computationally expensive initialization task. Perhaps you do some system checks when the program runs. Probably you don't want to run all those checks for every process you start. It can get even more interesting if you have values that change at runtime:

```python
import multiprocessing as mp
import random

val = random.random()

def simple_func():
    print(val)


if __name__ == '__main__':
    print('Before multiprocessing: ')
    simple_func()
    print('After multiprocessing:')
    p = mp.Process(target=simple_func)
    p.start()
    p.join()
```

On Windows, it would give an output like this:

```bash
Before multiprocessing:
0.16042209710776734
After multiprocessing:
0.9180213870647225
```

While on Linux, it gives an output like this:

```bash
Before multiprocessing:
0.28832424513226507
After multiprocessing:
0.28832424513226507
```

And this brings us to the last topic and the reason why I lost so much time when I had to port code written in Linux to work on Windows. A typical situation in which values change at runtime is when you are working with classes. Objects are meant to hold values; they are not static. So, what happens if you try to run a method of a class on a separate process? Let's start with a straightforward task:

```python
import multiprocessing as mp


class MyClass:
    def __init__(self, i):
        self.i = i

    def simple_method(self):
        print('This is a simple method')
        print(f'The stored value is: {self.i}')

    def mp_simple_method(self):
        self.p = mp.Process(target=self.simple_method)
        self.p.start()

    def wait(self):
        self.p.join()


if __name__ == '__main__':
    my_class = MyClass(1)
    my_class.mp_simple_method()
    my_class.wait()
```

The code works fine both on Linux and Windows. And this may happen for a lot of different scenarios, until one day you try to do something slightly more complicated, like writing or reading from a file:

```python
import multiprocessing as mp


class MyClass:
    def __init__(self, i):
        self.i = i
        self.file = open(f'{i}.txt', 'w')

    def simple_method(self):
        print('This is a simple method')
        print(f'The stored value is: {self.i}')

    def mp_simple_method(self):
        self.p = mp.Process(target=self.simple_method)
        self.p.start()

    def wait(self):
        self.p.join()
        self.file.close()


if __name__ == '__main__':
    my_class = MyClass(1)
    my_class.mp_simple_method()
    my_class.wait()
```

On Linux, the code above works fine. On Windows (and Mac), however, there'll be a very nasty error:

```bash
[...]
    ForkingPickler(file, protocol).dump(obj)
TypeError: cannot serialize '_io.TextIOWrapper' object
```

Pay attention to the fact that we don't do anything with the file. We just open and store it as an attribute in the class. However, the error already points to an interesting feature. The way Spawning works is by pickling the entire object. Therefore, if we have a class or an attribute that is not *pickable*, we will not be able to start a child process with it. 

And, for people working with hardware, most likely the communication with the device, in pretty much the same way that a file is non-pickable. It does not matter how much you try to make it multiprocessing safe by implementing locks or whatnot. The root problem is at a lower level. 

## Is there a way of solving it?
Sadly, there is no way of changing how processes start on Windows. You can, on the other hand, change how processes start on Linux. It would allow you to be sure your program also runs on Windows and Mac. We just need to add the following:

```python
if __name__ == '__main__':
    mp.set_start_method('spawn')
    my_class = MyClass(1)
    my_class.mp_simple_method()
    my_class.wait()
```

By using ``set_start_method``, the program will give the same error on Windows and Linux. Whether you need to add this line or not depends on what do you want to achieve. 

So, if you ever encounter these discrepancies, you will have to re-think the design of your program. I had objects with non-pickable attributes, especially drivers for devices and ZMQ sockets. 

## Speed is another factor
Even though processes usually speed up the speed of a program by leveraging multiple cores on a computer, starting each process can be time-consuming. The fact that on Windows and Mac Python needs to *pickle* the objects to create child processes adds an overhead that may offset the benefits of running on separated processes. It is especially relevant when you have many small tasks to perform, instead of a couple of long-running ones. 

Therefore, when using processes, improving the speed of the program is not a granted outcome. You should always benchmark your application to understand where and how different components can affect its behavior. 

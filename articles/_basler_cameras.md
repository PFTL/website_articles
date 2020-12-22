# Acquiring images from Basler Cameras

## Installing
Basler cameras require two components to work with Python. First, the [Pylon software suite](https://www.baslerweb.com/en/sales-support/downloads/software-downloads/#type=pylonsoftware;language=all;version=all), which includes Pylon Viewer, a program to quickly acquire images from the cameras, and also the libraries to build custom software. In the download page, be sure to select the package for your operating system and architecture. 

After installing Pylon, we can start the Pylon Viewer. This program allows us to test the camera, acquire images, and configure it. Testing whether we can acquire images is particularly important for Linux users, since accessing USB ports may require additional permissions depending on the system configuration. Also, the Pylon Viewer is a great starting point for finding out the name of the functions and parameters we may need to change in our programs. 

After Pylon is installed and working, we must install PyPylon, which is the Python wrapper for the Pylon SDK. We must get a release from the [Github repository](https://github.com/Basler/pypylon/releases). Bear in mind that PyPylon sometimes lags behind regarding Python releases. Be sure you are using a version of Python compatible with the PyPylon wheel available. To install the release, we can simply run the following (ideally inside a virtual environment):

```bash
pip install <pylon_wheel>.whl
```

### Problems with PyPylon installation
I have found that sometimes there is a mismatch between the latest Pylon version and the released PyPylon wheel available to download. Even if this does not raise problems during installation, it may be that we face an error like the following when running the examples of this tutorial:

```bash
ImportError: cannot import name '_pylon' 
```

The best solution is to [download the PyPylon code](https://github.com/basler/pypylon), and install it locally by running:

```bash
$ python setup.py install
```

Depending on your operating system, there may be another error associated with it. In Linux, Pylon is expected in the folder ``/opt/pylon5``, but newer versions of Pylon are installed on ``/opt/pylon``. The simplest solution is to create a system variable to point the installer to the right folder:

```bash
$ export PYLON_ROOT=/opt/pylon
$ python setup.py install
```

## Getting Started
When getting started, we must make sure that we can identify the camera we want to work with. This is not only useful for practical matters, but it is also useful to start understanding the patterns that the drivers require. We can run the following code:

```python
from pypylon import pylon


tl_factory = pylon.TlFactory.GetInstance()
devices = tl_factory.EnumerateDevices()
for device in devices:
    print(device.GetFriendlyName())
```

Which will generate a list of the cameras connected to our computer, similar to what we see with the PylonViewer program. In my case, the output is:

```bash
Basler a2A1920-160umBAS (40063823)
```

The code above starts by creating a *transport layer* instance, and enumerating the devices it has access to. According to the manual, the transport layer is an abstraction of the physical connection between the camera and the computer (i.e. USB, Camera Link, etc.). Its main purpose is to find and manage the life cycle of the cameras. The method ``EnumerateDevices`` returns a tuple containing information of each connected camera. 

Bear in mind that PyPylon is built using Swig, a library that allows to wrap C code into Python automatically. If we would print ``device`` instead of its friendly name, we would get an output like this:

```python
"<pypylon.pylon.DeviceInfo; proxy of <Swig Object of type 'Pylon::CDeviceInfo *' at 0x7fb7c3cd53f0> >"
```

Which is not particularly useful. Each object returned by the enumeration of the transport layer allows for the unique identification of the camera. This means that the information contained within it is all we need to start communicating with a camera. Other methods that may be useful to identify a camera are: ``GetFullName()`` which returns a unique code like ``2676:ba05:3:2:10``, or ``GetSerialNumber()`` which returns a number like ``40063823``. 

In many cases we have only one camera, and the rest of the article will proceed on that assumption. If it is not the case, we can address the camera we want by using its unique properties. 

## Acquiring an Image
If we are working with a camera, the first thing we may want to do is to acquire an image. We can create an ``InstantCamera`` object by running the following code:

```python
tl_factory = pylon.TlFactory.GetInstance()
camera = pylon.InstantCamera()
camera.Attach(tl_factory.CreateFirstDevice())
```

Bear in mind that we are using the first device, whichever it may be. When working with several cameras, it may be wiser to use ``CreateDevice`` instead. 

It is important to note in the code above that there are two steps involved in getting started with a Basler camera. First, the transport layer is responsible for actually creating the device. The InstantCamerarigni
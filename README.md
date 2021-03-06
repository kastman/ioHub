# ioHub

# Project Status

The ioHub is actively in development, but is very new. It is therefore
at a point where it is very likely that downloading the code and trying
to run it will fail, or that not all features are fully functional at this
time. Please bear with us as we get the ioHub to a stable 'alpha' state.

If you are looking to view the state of the source for the pyEyeTrackerInterface,
it is in the ioHub.devices.eyeTrackerInterface folder of the module. The above 
comments apply to it as well at this time. 

#Overview

The ioHub module is intended to be a standalone Python service that runs
in parallel to psycopy ( http://www.psychopy.org ) during experiment 
runtime / data collection. The ioHub can also be used by any other
application that is interested in the 'service' it offers.

The ioHub effectively acts as a proxy between the experiment logic 
and the input and output devices used by the experiment, centralizing 
a large portion of the I/O bound tasks an application may have into a
common asynchronous non-blocking architecture that runs in a seperate
independent process than the application itself (not a child process).

The **main features / goals** of the ioHub are:

1. **Integrate data collection from multiple devices**, including keyboard, 
   mouse, parallel port, eye trackers ( using the Common Eye Tracker 
   Interface project ), etc.
   
2. **Time Base Syncronization.**

2a. For devices that provide data streams and / or events that are time
   stamped, the ioHub will convert the various "Device Time"s to a common
   "Application Time" base. The exact mechanism for determining the 'offset' 
   between the two time bases is device dependent, please see the full 
   documentation for details. The base offset that is determined is applied
   to convert the Device Time to Application Time prior to further corrections.

2b. For devices that do not provide time stamped data streams or events, the ioHub
   will time stampe them in Application Time when the ioHub receives the event.
   An important goal of the ioHub is to keep its core IOLoop as fast as possible, 
   following a non-blocking asyncronous methodology whenever possible. 
   This has the effect of it being able to check for polled device updates 
   quickly ( currently several times a msec ). This also means time stampes
   for this type of devices whould be sub millisecond *realative to when 
   the event was received by the hub*. 

2c. The accuracy and precision of time stamping is
   important to the ioHub, so it does what it can, when it can:*  

   The offset between time bases is corrected for, when an existing 
   time stamp is present.

   Delay that is measurable, or a known average, can be applied to
   each device data stream and event type to correct for the delay
   in the time stamp.

   Drift between the Application Time base and each Device's timebase
   can be actively monitored and also corrected for in the Application
   time stamping. This is necessary when the Application Time base and
   the Device Time are derived from difference clocks / crystals.

   *It is important to note that the ability for the ioHub to correct 
   for the above factors is 100% device and OS dependent. If a device
   has not been designed with proper time base interfacing in mind, and
   there are therefore limited, coarse, or no ways to determine one or more
   of these factors, then the ioHub can only do so much in this area.
   The documentation will have a section outlining what is in place for
   each device, what the level of expected accuracy and precision should be, 
   and what (if any) tests have been done to date to validate the 
   time base corrections.

3. **Common Data Stream / Event Access and Data Types**, regardless of device. 
   The ioHub, while normalizing the time stamps of all input events to a 
   common experiment / application level timebase, also provides the 
   convenience of a single interface to device data, and common device
   sample and event definition standards. Furthermore devices within
   the same device category will have their sample data and events mapped
   to a single set of vendor independent structures as much as possible.

4. **Low Overhead Design.** The ioHub runs as a seperate process from your
   experiment / psychopy, while at the same time doing work that your application
   once needed to spend CPU time on and perhaps dead I/O blocking time on
   (depending on your application design of course). The ioHub allows 
   for this work to be done truely in parallel with your application on 
   any multiprocessor / multicore based computer. Multicore CPUs are now
   standard for laptop and desktop computers. These advantages are provided 
   with a simple and fast request / response IPC architechure using standard 
   Python UDP sockets / packets, making it very cross platform to boot.
   The request / response pattern also hels ensure that, if a UDP packet
   is dropped, the ioHub Client, or the ioHub itself will know about
   it and can handle it. Current performance tests on Windows 7, using an
   i5 mobile chipset connecting with 127.0.0.1, show base request / response 
   round trip times through the core ioHub infrastructure taking under 
   300 usec on average ( 0.3 msec, 0.003 sec ) with packet loads of about 1400 bytes. 
   In all tests preformed to date, no dropped packets have occurred.

5. **Data Storage.** Given all the data stream / event based processing that
   the ioHub is doing, it seems to only make sense to offer the option for
   it to also save all this data from the ioHub process, further reducing
   the overhead and processing required by your application if it is not 
   needed. The experiment runtime can therefore request certain events to
   be available over the UDP connection, while all data be saved to disk.
   The experiment itself can in effect act as an input device in this case,
   sending Experiment Events to the ioHub to be integrated with the rest 
   of the device data and saved for future analysis. pyTables will be used
   as the API / storage mechanism for the ioHub. This functionality is packaged
   the ioHub.ioDataStore directory. The use of this functionality is completely optional,
   may not be of use in all situations, and requires further requirements refinement refinement
   to move implementation beyond an alpha stage of functionality. With that said, it also has a huge 
   amount of potential.

# Installing

  Currently only Windows is supported. This will change to Linux and Mac OS X as well
  as time permits.
  
  Python 2.7.3 32 bit for Windows is required as the python interpreter (not Python 3.0, 
  and not Python 2.6 due to a couple bugs, that could be worked around, but have yet to be)
  (can be installed on 32 or 64 bit version of OS)

  You then need to install psychopy and all of it's dependencies. Note that the 'all in one' Windows
  installer installs 2.6 at this time, so it can not be used. You must install psychopy as a 
  package and all the psychopy dependencies seperately.
  
   psychopy 1.74.01 for Python 2.7 - http://code.google.com/p/psychopy/downloads/

  And all dependencies for it, which are listed at:

   http://www.psychopy.org/installation.html#dependencies

  You will want to get with python 2.7 win32 version of each dependency.
 
  **For numpy, please get 1.6.2 or higher.**

  OK, once you have all that installed, you will want to get the following extra 
  dependencies for ioHub and ioDataStore:
  
   psutil: http://code.google.com/p/psutil/downloads/detail?name=psutil-0.5.1.win32-py2.7.exe
 
   ujson: http://pypi.python.org/packages/2.7/u/ujson/ujson-1.19.win32-py2.7.exe#md5=a5eda15e99f6091e9e550887b35e7fd4

   msgpack: http://pypi.python.org/packages/2.7/m/msgpack-python/msgpack_python-0.2.0-py2.7-win32.egg#md5=d52bd856ca8c8d9a6ee86937e1b4c644

   gevent: http://code.google.com/p/gevent/downloads/detail?name=gevent-1.0b3.win32-py2.7.exe&can=2&q=
   
   greenlet: http://pypi.python.org/packages/2.7/g/greenlet/greenlet-0.4.0.win32-py2.7.exe#md5=910896116b1e4fd527b8afaadc7132f3

   pytables: http://www.lfd.uci.edu/~gohlke/pythonlibs/#pytables 
    
   numexpr: http://code.google.com/p/numexpr/downloads/detail?name=numexpr-1.4.2.win32-py2.7.exe&can=2&q=

   pywin32: http://sourceforge.net/projects/pywin32/files/pywin32/Build%20217/pywin32-217.win32-py2.7.exe/download
    
   pyHook: http://sourceforge.net/projects/pyhook/files/pyhook/1.5.1/pyHook-1.5.1.win32-py2.7.exe/download
   
   'Finally', the ioHub source to your site-packages folder, putting the ioHub directory in the site-packages directory 
   of your Python 2.7 installation. 

# Known Issues / Black Holes

See the Bug Tracker.

# Getting Help

email: sds-git _AT_ isolver-software.coT (change _AT_ to an @ and the T to an m)

# License

ioHub and ioDataStore are Copyright (C) 2012 Sol Simpson, except for files where otherwise noted.

ioHub and ioDataStore are distributed under the terms of the GNU General Public License (GPL version 3 or any later version).
See the LICENSE file for details.


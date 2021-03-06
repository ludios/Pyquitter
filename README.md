Pyquitter
=========

So, you've got a program that you're working on, and you're tired of
hitting Ctrl-C, Up, Enter.  You'd like it to just restart every time you
make a change.  Here's how Pyquitter helps you solve this problem:

1.	Early in your program startup, you instantiate
	`pyquitter.detector.ChangeDetector` with a 0-arg callable
	(one that quits your program).

2.	In addition, you set up a timer to call `yourChangeDetector.poll()`
	every few seconds.  If any of your imported modules have changed,
	the `callable` you passed in earlier will be called.

3.	Your callable quits your program. (`sys.exit()`, or if using Twisted, `reactor.stop()`)

4.	A parent process keeps restarting your program in a loop.  You can use
	the included `bin/looper` if it suits your needs.



## Installation

```
git clone https://github.com/ludios/Pyquitter
cd Pyquitter
pip install --user .
```

This installs the module `pyquitter` as well as the binaries `looper` and
`looper-stop`.

Pyquitter is also available on PyPI <http://pypi.python.org/pypi/Pyquitter>
and you can install it with pip:

```
pip install --user Pyquitter
```



## Sample use

This example uses Twisted, but there's nothing Twisted-specific in Pyquitter.

```py
# demo.py

import sys
from twisted.python import log
log.startLogging(sys.stdout)

from twisted.internet import task
from twisted.internet import reactor
from pyquitter.detector import ChangeDetector

stopper = ChangeDetector(
	lambda: reactor.callWhenRunning(reactor.stop),
	logCallable=log.msg)
	# logCallable is optional; if not passed, it uses print.

looping = task.LoopingCall(stopper.poll)
looping.start(1.0, now=True) # check every 1 sec

reactor.run()
```

Run the above program with `looper`, which will restart it every time it quits:

```
looper python demo.py
```

or on Windows:

```
C:\Python27\Scripts\looper C:\Python27\python demo.py
```

(Note that in the normal `cmd` shell, the above command actually runs `looper.bat`.)

If you modify demo.py or any module it has imported, you'll see the program
restart.  If a module is modified but there are syntax errors, the callable
passed to `ChangeDetector` will not be called until the syntax errors are
fixed.



## Watching for changes to non-module files

After `stopper = ` in the above example:

```py
stopper.watchNonModuleFile('/home/me/my_settings.py')
stopper.watchNonModuleFile('/home/me/blob')
```

Note that because `/home/me/my_settings.py` ends with '.py', the
file will be checked for syntax errors using Python's `compile`
function.  As with module files, if there are syntax errors,
`callable` will not be called until the errors are fixed.

To stop watching a non-module file:

```py
stopper.forgetNonModuleFile('/home/me/my_settings.py')
```



## Running the tests

Install Twisted, then `trial pyquitter`.



## Wishlist

*	Detect changes to modules outside of the process.  Feed a list of files
	to some parent process that monitors the modules for the often-restarting child.
	This would fix a race condition where a module is changed before
	`ChangeDetector` takes a look at it.

*	Rewrite the tests; don't spawn child `python`s.

*	Document optional use of Pyflakes (see the source for now).

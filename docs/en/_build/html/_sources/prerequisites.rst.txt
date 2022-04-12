==============
Pre-requisites
==============

OS Recommendations
------------------

The charts below show how Hyperchain's requirements map onto various
platforms.

**Platforms** 

+--------+----------------+------------+
| Distro |    Release     |    Arch    |
+========+================+============+
| RHEL   | 6 or later     | amd64, 386 |
+--------+----------------+------------+
| CentOS | 6 or later     | amd64, 386 |
+--------+----------------+------------+
| SLES   | 11SP3 or later | amd64, 386 |
+--------+----------------+------------+
| Ubuntu | 14.04 or later | amd64, 386 |
+--------+----------------+------------+
| macOS  | 10.8 or later  | amd64, 386 |
+--------+----------------+------------+

Install Go
----------

Hyperchain uses the Go programming language for its components, thus we
need to install Go for developing.

Download Go
```````````
Go provides binary distributions for Mac OS X, Linux,
and Windows. If you are using a different OS, you can download the Go
source code and install from source.

Download the latest version of Go for your platform here: 
`Downloads <https://golang.org/dl>`__ - version ``1.7.x or above``

Install Go
``````````
Follow the instructions for your platform to install the
Go tools: `Install the Go
tools <https://golang.org/doc/install#install>`__. It is recommended to
use the default installation settings.

-  On Mac OS X and Linux, by default Go is installed to directory
   ``/usr/local/go/``, and the ``GOROOT`` environment variable is set to
   ``/usr/local/go``.

.. code:: bash

    export GOROOT=/usr/local/go

-  Also set the ``GOROOT/bin`` variable, which is used to run Go
   command.

.. code:: bash

    export PATH=$PATH:$GOROOT/bin

Set GOPATH
``````````
Your Go working directory (``GOPATH``) is where you store
your Go code. It can be any path you choose but must be separate from
your Go installation directory (``GOROOT``).

The following instructions describe how to set your ``GOPATH``. Refer to
the official Go documentation for more details:
https://golang.org/doc/code.html.

-  On Mac OS X and Linux Set the ``GOPATH`` environment variable for
   your workspace:

.. code:: bash

    export GOPATH=$HOME/go

-  Also set the ``GOPATH/bin`` variable, which is used to run compiled
   Go programs.

.. code:: bash

    export PATH=$PATH:$GOPATH/bin

-  Since we'll be doing a bunch of coding in Go, you might want to add
   the following to your\ ``~/.bashrc``:

.. code:: bash

    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$PATH:$GOPATH/bin:$GOROOT/bin

Test Go Installation
````````````````````
Create and run the hello.go application
described here: https://golang.org/doc/install#testing.

If you set up your Go environment properly, you should be able to run
"hello" from any directory and see the program execute successfully.

Install Go vendor tool
----------------------

Go vendor is a tool for managing Go packages and their dependencies.
This tool will copy the dependent packages to the project's ``vendor``
directory, and record their versions in a file named ``vendor.json``.

Installation
````````````
.. code:: bash

    go get -u github.com/kardianos/govendor

Test Go vendor Installation
```````````````````````````
To verify you setup govendor properly,
please make sure govendor version information displays correctly.

At the command prompt, type the following command and make sure you see
govendor version information:

.. code:: bash

    $ govendor --version
    v1.0.9

More Details
````````````
You can goto the project's home page for more details.
- `Go vendor <https://github.com/kardianos/govendor>`__

Install Contract Compiler(Optional)
-----------------------------------

Hyperchain supports Smart Contract which written in
`Solidity <https://solidity.readthedocs.org/en/latest/>`__ and then
compiled into bytecode to be uploaded on the blockchain.

Given that we are writing in Solidity, we need to be sure that we have
installed contract compiler named ``solc`` for compiling.

We've provided some general installers for some platforms in our source
code, you can use them to install ``solc`` quickly, you can also refer
to the official site - `Installing
Solidity <https://solidity.readthedocs.io/en/latest/installing-solidity.html#installing-solidity>`__
for installation.

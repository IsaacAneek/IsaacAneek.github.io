How to set up AVR Toolchain for Lite XL code editor (Windows)
=============================================================
Backstory
---------
A couple of days ago I was trying to program my Arduino UNO R3 board using my good ol' laptop with only 4Gigs of RAM and Dual Core Processor.
Previously I had tried VS Code and Arduino IDE but it definitely wasn't the smoothest experience. They are built on the Electron framwork so they
aren't supposed to run smooth on low-end devices either. I was searching for a lightweight code editor and yes that's when found about Lite XL text editor.

The bundle size is small (around 5MB) and yea it runs crispy smooth on my good ol' laptop. It's also extensible with plugins. Now let's get started.

Lite XL
--------
You can start by downloading and installing Lite XL from `their website`_

.. _their website: https://lite-xl.com/

Open a *.c or *.py file and you are good to go. Yes there will be lot of errors because the editor doesn't support autocomplete and linting
out of the box. It only comes with syntax highligting and some basic features. We need to install plugins to leverage the advanced features.

Installing Lite XL Plugin Manager
----------------------------------
Lite XL doesn't come with the Plugin Manager by default. Install it `from here`_ and restart the editor.

.. _from here: https://github.com/lite-xl/lite-xl-plugin-manager#installing

To verify if it has been succesfully installed, open the editor and tap ``ctrl + shift + p`` and search for "Plugin". If it appears then
you are good to go.

Downloading the AVR Toolchain and related tools
-----------------------------------------------
These are some good resources to learn how to download AVR Toolchain and related tools.

* `AVR GCC Toolchain – Setup for Windows <https://tinusaur.com/guides/avr-gcc-toolchain/>`__
* `How to setup the AVR toolchain on Windows <https://www.tonymitchell.ca/posts/setup-avr-toolchain-on-windows/>`__

.. note::
    It's not necessary  to download and install all the tools mentioned. The required tools are:

    - AVRDUDE
    - AVR Toolchain
    - GNU Make
    - CMake (Not mentioned but we will need it later)

.. important::
    Make sure to add the directory of each tool to the PATH variable. This will allow windows  to search the paths specified
    in the PATH variable to find the corresponding executable whenever we execute a CLI in the command prompt.

Building a simple blinky.c example manually
--------------------------------------------
Create a new project by following `this <https://www.tonymitchell.ca/posts/building-avr-projects-with-make/#project-setup>`__ .
Open the blinky.c with Lite XL. The editor might show a lot of errors but for now we are gonna ignore that. We will try build the project
manually and see if we can succeed. `Follow this <https://www.tonymitchell.ca/posts/building-avr-projects-with-make/#building-it-manually>`__ and 
it should compile without errors. If it does then we are good to go for the next step.

Configuring LSP Plugin for Lite XL editor
-------------------------------------------
Now the real pain begins.

    Plugin that provides intellisense for Lite XL by leveraging the LSP protocol

from their `doc <https://github.com/lite-xl/lite-xl-lsp>`__. To install LSP:

- Open Lite XL
- Press ``ctrl + shift + p``
- Search for ``Plugin Manager: Show``
- Press and again search for ``Language Server Protocol`` and ``lsp_c``. Install both. 
- Or you can manually install the LSP plugin `from here <https://github.com/lite-xl/lite-xl-lsp>`__. 

Restart editor and you should be able to see smooth autocompletion?? No. Instead we will see heaps of red lines and errors.
Now there are a couple of reasons.

- The Language Server (more specifically clangd) doesnt know where to search for header files since we are using custom toolchain.
- Some files in the AVR Toolchain contains undefined macros. This is ok since ``avr-gcc`` defines those macros at compile time [1]_. but
  since clangd doesn't know this [2]_ and it emulates clang, it will show errors. They typically look like ``invalid preprocessor directive``

Now there are a couple of ways we can mitigate these issues.

- Using a global ``clang.yaml`` config file to configure clang. [3]_
- Creating and using project specific ``compile_commands.json`` using ``CMake`` [4]_

The first one is fast but temporary solution. It solves the ``header file not found`` problem by specifying compile flags to search
for specified path for header files. Also it configures the clangd Language Server globally. This might create issues if we are working
on multiple projects with multiple different toolchains.

So we will be implementing the second solution as it is more robust.

Creating and using project specific ``compile_commands.json`` using CMake
--------------------------------------------------------------------------
Download the CMakeLists.txt file from `here <https://github.com/IsaacAneek/cmake-make-avr/tree/main/CMake>`__ and put it in your project
directory. Open it. We are gonna change only a couple of things and leave the rest as is.

- ``set(PROGRAMMER_ARGS -P COM7)``.
  Change the ``COM7`` to whataver the port your board is using. Use Device Manager if you don't know the
  port.
- ``project (main C CXX ASM)``
  Change ``main`` to name of your project file. In our case it would be ``blinky``
- ``set(INC_PATH     "D:/Softwares(Setup)/AVR Toolchain/avr8-gnu-toolchain-win32_x86_64/avr/include")``
  Replace this with your own ``....avr8-gnu-toolchain-win32_x86_64/avr/include`` directory

Ok we are done for now.

Now we will try to create ``compile_commands.json``. This file configures clangd to search for specified paths for headers.
It also mitigates the undefined macro problems. You can create ``build`` directory to seperate your src code
and your build code but for now will be working on our project directory. Open PowerShell/CMD in your project directory and run this command

.. code-block::

    cmake -G "MinGW Makefiles" -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

This will create a ``MakeFile`` and a ``compile_commands.json`` in the project diretory.
Now if you restart the editor and open the blinky.c file again there should be no errors.
The autocomplete should work properly. PORT variables should come up in the autocomplete suggestions. 
No ``invalid preprocesor directive`` error should come up.

Building the code and flashing it to the UNO R3
------------------------------------------------
Open PowerShell/CMD in the project directory again.
Run

.. code-block::

    make all

This should compile and link your project files without any errors.

Then run this to flash the *.hex file to the UNO R3 board.

.. code-block::

    make flash

The flash should complete without any errors.

References
----------
.. [1] https://gcc.gnu.org/onlinedocs/gcc-8.3.0/gcc/AVR-Options.html#AVR-Built-in-Macros
.. [2] https://github.com/clangd/clangd/issues/528#issuecomment-695356278
.. [3] https://clangd.llvm.org/config.html#compileflags
.. [4] https://clang.llvm.org/docs/JSONCompilationDatabase.html
Autotools Tutorial
==================

This provides a tutorial on how to use **autotools**. Autotools provides a set of programs that allow you to create a ``configure`` script, which many linux users are familiar with as the common way to build apps from source: ``./configure && make && make install``.  
Configure scripts are used for any decently sized program to ensure that the app is compatible with a larger set of system configurations.

The example C program is a simple webkit browser written in C that uses the ``webkit`` and ``gtk+ 2.0`` libraries. The source file is ``cwebkit.c``. This tutorial is based on ``autoconf v2.69``.



Preliminary Instructions
------------------------

Before diving into the **autotools** tutorial, it's a good idea to first test drive the actual program. The program itself requires the ``webkit-1.0`` and ``gtk+-2.0`` libraries. 

You might already have these libraries installed. Use ``pkg-config`` to check if you have them installed:

```
pkg-config --list-all | grep 'webkit-1.0\|gtk+-2.0'
```

The ``pkg-config`` program should show that you have both ``webkit-1.0`` and ``gtk+-2.0`` installed. If you don't, you'll need to install them on your system. In Fedora, for example, the packages are called ``webkitgtk-devel`` and ``gtk2-devel``. In Arch Linux they're called ``webkitgtk`` and ``gtk2``. In Ubuntu they're called ``libwebkit-dev`` and ``libgtk2.0-dev``, and so on...

Once you've determined that you have the correct libraries installed, rename ``Makefile.original`` to ``Makefile`` in the ``src`` directory, and run ``make`` to build the program. The program should build correctly. Then revert the filename back to ``Makefile.original`` so as to not conflict with autotools in later steps.

Run ``./cwebkit`` in the ``src`` directory to test out the program. It takes a minute for the web page to load. You should see the [DuckDuckGo](http://www.duckduckgo.com) main page, indicating that the browser app worked.

Open up the ``cwebkit.c`` and ``Makefile`` files to get a feel for how this simple program is structured, and then move on to the **autotools** tutorial instructions in the next section.


Instructions
------------

1. Check that you have autotools installed.  
Check that the programs ``autoscan``, ``autoconf``, ``aclocal``, and ``automake`` are installed by typing, for example, ``whereis autoscan``. 
Also, if you skipped the preliminary instructions, now is a good time to make sure that you have webkit and gtk installed.

2. View the ``Makefile.am`` files.  
The automake Makefiles ``Makefile.am`` and ``src/Makefile.am`` contain the necessary information required for the ``automake`` program to create the ``Makefile.in`` versions of the Makefiles. The ``Makefile.in`` versions of the Makefiles are what's used by the ``configure`` script to generate the core ``Makefile`` that's used to run ``make`` and ``make install``.  
In the core directory, ``Makefile.am`` contains two lines only:
```
AUTOMAKE_OPTIONS = foreign
SUBDIRS = src
```
The first line, ``AUTOMAKE_OPTIONS``, tells ``automake`` to disable many of GNUs annoying warning messages that are unnecessary for this tutorial.  
The second line, ``SUBDIRS``, tells ``automake`` to include the subdirectory called ``src``, which contains the second and main Makefile.
In the ``src`` directory, the ``Makefile.am`` contains the following:  
```
AM_CFLAGS=-pthread -I/usr/include/webkitgtk-1.0 -I/usr/include/gtk-2.0 -I/usr/lib64/gtk-2.0/include -I/usr/include/pango-1.0 -I/usr/include/atk-1.0 -I/usr/include/cairo -I/usr/include/pixman-1 -I/usr/include/libdrm -I/usr/include/libpng16 -I/usr/include/gdk-pixbuf-2.0 -I/usr/include/libpng16 -I/usr/include/pango-1.0 -I/usr/include/harfbuzz -I/usr/include/pango-1.0 -I/usr/include/freetype2 -I/usr/include/libsoup-2.4 -I/usr/include/libxml2 -I/usr/include/webkitgtk-1.0 -I/usr/include/glib-2.0 -I/usr/lib64/glib-2.0/include
AM_LDFLAGS=-lwebkitgtk-1.0 -lgtk-x11-2.0 -lgdk-x11-2.0 -lpangocairo-1.0 -latk-1.0 -lcairo -lgdk_pixbuf-2.0 -lpangoft2-1.0 -lpango-1.0 -lfontconfig -lfreetype -lsoup-2.4 -lgio-2.0 -lgobject-2.0 -ljavascriptcoregtk-1.0 -lglib-2.0
bin_PROGRAMS = cwebkit
cwebkit_SOURCES = cwebkit.c
```
The first two lines are identical to the original Makefile ``Makefile.original``s ``CFLAGS`` and ``LDFLAGS`` (when ``pkg-config`` is run on them), except that they are prefixed with ``AM_`` so that ``automake`` will properly process them.  
The ``CFLAGS`` are dervied by running ``pkg-config --cflags webkit-1.0 gtk+-2.0``, and the ``LDFLAGS`` are derived by running ``pkg-config --libs webkit-1.0 gtk+-2.0``.  
The ``bin_PROGRAMS`` line tells ``automake`` which binary files it should make. In the original Makefile ``Makefile.original``, this is equivalent to the ``cwebkit:`` line.  
The ``cwebkit_SOURCES`` line tells ``automake`` which source files it should use to make the binary file ``cwebkit``. In this simple example, ``cwebkit.c`` is all that's needed to make the binary file ``cwebkit``, but this line could also include header files or other source files that are necessary to create the binary file.

3. Run ``autoscan`` to generate an autotools configuration file called ``configure.scan``.  
The ``autoscan`` program will automatically pick up information from ``cwebkit.c`` and the two ``Makefile.am`` files to fill in necessary info for the configuration file. The ``autoscan`` utility isn't perfect, however, and can miss many things, which is why the configure file needs to be manually edited to ensure that the final ``configure`` script is robust.

4. Rename ``configure.scan`` to ``configure.ac`` so that the ``autoconf`` program can recognize it.

5. Edit ``configure.ac`` and modify it as follows:  
  1. Remove the ``AC_CONFIG_HEADERS`` line. This simple tutorial doesn't have (or need) a corresponding ``config.h`` file.  
  (See the [autoconf manual](https://www.gnu.org/software/autoconf/manual/autoconf-2.65/html_node/Configuration-Headers.html) for more details on writing a ``config.h`` file).

  2. Replace the ``AC_INIT`` line with metadata about the program:  
  ``AC_INIT([cwebkit], [0.1], [risto1@gmail.com])``
  
  3. Fix the library checks.  
  The ``AC_CHECK_LIBS`` lines will ensure that when ``./configure`` is called, it doesn't fail in the build step when ``make`` is called and the user doesn't have a necessary libraries.  
  You may have had the frustrating situation when you run the ``./configure && make && make install`` on an application you're trying to build from scratch and either the ``./configure`` script fails or the build fails. This is usually due to an improperly written ``configure`` script.  
  One very easy and important step to avoid this is to ensure that your ``configure`` script checks that all library dependencies are installed. You should offer a descriptive error message to the user rather than a default (and often *cryptic*) one when the user is in the middle of building your program.  
  ``autoconf`` picked up some dependencies but not all of them. Modify the ``AC_CHECK_LIBS`` lines with the following:
```
AC_CHECK_LIB([webkitgtk-1.0], [webkit_web_view_load_uri], [], [
              echo "The webkit library is required for this program."
              exit -1])
AC_CHECK_LIB([gtk-x11-2.0], [gtk_init], [], [
              echo "The gtk library is required for this program."
              exit -1])
AC_CHECK_LIB([cairo], [cairo_set_font_size], [], [
              echo "The cairo library is required for this program."
              exit -1])
AC_CHECK_LIB([fontconfig], [FcInitLoadConfig], [], [
              echo "The fontconfig library is required for this program."
              exit -1])
AC_CHECK_LIB([freetype], [FT_Init_FreeType], [], [
              echo "The freetype library is required for this program."
              exit -1])
```
The first parameter for ``AC_CHECK_LIB`` is the name of the library to check. To see all libraries required by this program, type in ``pkg-config --libs webkit-1.0 gtk+-2.0``. Not all of the required libraries are here for the sake of simplicity, but you probably want to add them all if you're writing an application for other people to use (rather than have it break in the middle of the build for some poor user).  
The second parameter requires you to specify a function that exists in that library. In the fourth parameter, you need to put the commands on separate lines so that the ``configure`` script can properly exit when a library is not found.  
The signature for this macro is:  
``AC_CHECK_LIB (library, function, [action-if-found], [action-if-not-found], [other-libraries])``  
More info on ``AC_CHECK_LIB`` can be found in the [autoconf manual](http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/Libraries.html).  
See (or diff) the ``configure.ac.sample`` file to make sure that you edited everything correctly.

4. Run ``autoconf`` to generate the first state of the configuration script.

5. Add ``AM_INIT_AUTOMAKE`` before the ``AC_OUTPUT`` line.  
This macro is required to invoke ``automake``. It tells ``automake`` to generate Makefiles after the ``configure`` script is run.  
If you don't put the line before ``AC_OUTPUT``, you will get this error message when building:  
``Error: Makefile: : *** missing separator. Stop.``.

6. Run:  
```
aclocal
autoconf
automake -a
```
The ``aclocal`` program creates an ``aclocal.m4`` file that's used by ``autoconf``.  
Rerunning ``autoconf`` with the ``AM_INIT_AUTOMAKE`` macro line and the ``aclocal.m4`` file in place ensures that the ``configure`` script is finally ready. If you need to redo this step or change ``configure.ac``, you can run ``autoreconf`` to rerun all necessary steps.  
The ``automake -a`` line generates the ``Makefile.in`` files that are used by the ``configure`` script to create the real ``Makefile`` file that's used when ``make`` and ``make install`` are invoked. 


That's it! now you can test your configure script:  
```
./configure && make && make install
```
or if you are using sudo:
```
./configure && make && sudo make install
```

The binary files are normally installed under ``/usr/bin/local`` for linux systems. This path is usually included in the ``$PATH`` variable. You should therefore be able to run the program simply by typing ``cwebkit``!

To uninstall the build for whatever reason, you can just type:
```
sudo make uninstall
```

And if you want to install it in a different directory, say ``/home/myusername/cwebkit``, you can do:
```
./configure --prefix=/home/myusername/cwebkit && make && make install
```

Or if you just want it to install in the current folder, then skip the ``make install`` part of the process.

And finally, if you want to create a distribution ``.tar.gz`` package file for others to use, you can type:
```
make dist
```

That's it! I hope you found this tutorial very helpful and informative!
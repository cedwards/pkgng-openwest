=============================================
Next Generation Package Management in FreeBSD
=============================================

 - OpenWest 2013
 - May 2-4, 2013
 - Christer Edwards

History
=======

``pkgng`` is an improved replacement for the traditional FreeBSD package management
tools, offering many features that make dealing with binary packages faster and
easier. 

The first release of pkgng was in August, 2012.

What it is not
==============

 - a ports management tool (i.e. it doesn't know how to build a port)
 - a replacement for ``portupgrade``/``portmaster``

What it is
==========

 - a replacement for pkg_* tools
 - a tool to query/manage installed packages
 - a tool to deal with binary packages
 - a tool to upgrade/install packages from a remote repository
 - a library that provides all the package management in a safe way so one can write a new frontend

Getting Started (>=9.1)
=======================

FreeBSD 9.1 and later includes a "bootstrap" utility for ``pkgng``. The bootstrap
utility will download and install ``pkgng``.

To bootstrap the system, run

.. code-block:: bash

    # /usr/sbin/pkg

Getting Started (<9.1)
======================

For earlier FreeBSD versions, ``pkgng`` must be installed from the Ports
Collection, or as a binary package.

To install the ``pkgng`` port, run:

.. code-block:: bash

    # make -C /usr/ports/ports-mgmt/pkg install clean

To install the binary package, run:

.. code-block:: bash

    # pkg_add -r pkg

Conversion
==========

Existing FreeBSD installations require conversion of the pkg_install package
database to the new format. To convert the package database, run:

.. code-block:: bash

    # pkg2ng

This step is not required for new installations that do not have third-party
software installed.

.. admonition:: WARNING

    This step is not reversible. Once the package database has been
    converted to the pkgng format, the pkg_install tools should not be used.

Configuration
=============

To ensure the FreeBSD Ports Collection registers new software with ``pkgng``, and
not ``pkg_install``, FreeBSD versions earlier than 10.X require this line in
``/etc/make.conf``::

    WITH_PKGNG= yes

Configuring the pkgng Environment
=================================

The pkgng package management system uses a package repository for most
operations. The default package repository location is defined in
``/usr/local/etc/pkg.conf`` or the ``PACKAGESITE`` environment variable, which
overrides the configuration file.

**/usr/local/etc/pkg.conf:**

.. code-block:: bash

    PACKAGESITE: http://example.org/pkgng-repo/

**PACKAGESITE variable**

.. code-block:: bash

    # setenv PACKAGESITE http://example.org/pkgng-repo/
    # export PACKAGESITE=http://example.org/pkgng-repo/

Working with multiple repositories
==================================

It is possible to use multiple repositories with pkgng. In order to enable this
functionality, add ``PKG_MULTIREPOS : YES`` to ``/usr/local/etc/pkg.conf``.

.. code-block:: bash

    # echo "PKG_MULTIREPOS : YES" >> /usr/local/etc/pkg.conf

repos
=====

.. code-block:: bash

    repos:
      default : http://example.org/pkgng/
      repo1 : http://somewhere.org/pkgng/repo1/
      repo2 : http://somewhere.org/pkgng/repo2/

It is important that you always define a default repository - this is the
repository that is being used when no remote repositories are specified via the
-r <repo> flag.

And now you can install from the remote repositories using the pkg install
command like this:

.. code-block:: bash

    # pkg install -r repo1 py27-salt py27-salt-cloud

======================
Basic pkgng Operations
======================

Usage Information
=================

Usage information for pkgng is available in the ``pkg(8)`` manual page, or by
running ``pkg`` without additional arguments.

Each ``pkgng`` command argument is documented in a command-specific manual page. To
read the manual page for pkg install, for example, run either:

.. code-block:: bash

    # pkg help install
    # man pkg-install

Querying Installed Packages
===========================

Information about all installed packages is available by running:

.. code-block:: bash

    # pkg info

Information about a specific package is available by running:

.. code-block:: bash

    # pkg info packagename

Querying Installed Packages (Example)
-------------------------------------

For example, to see which version of pkgng is installed on the system, run:

.. code-block:: bash

    # pkg info pkg
    pkg-1.0.2           New generation package manager

Installing Packages
===================

Most FreeBSD users will install binary packages by running:

.. code-block:: bash

    # pkg install packagename
    # pkg install packageorigin

Installing Packages (Example)
-----------------------------

.. code-block:: bash

    # pkg install curl
    Updating repository catalogue
    Repository catalogue is up-to-date, no need to fetch fresh copy
    The following packages will be installed:

    Installing ca_root_nss: 3.13.5
    Installing curl: 7.24.0

    The installation will require 4 MB more space

    1 MB to be downloaded

    Proceed with installing packages [y/N]: y
    ca_root_nss-3.13.5.txz      100%    255KB   255.1KB/s 255.1KB/s 00:00
    curl-7.24.0.txz             100%    1108KB  1.1MB/s 1.1MB/s     00:00
    Checking integrity... done
    Installing ca_root_nss-3.13.5... done
    Installing curl-7.24.0... done

Removing Packages
=================

Packages that are no longer needed can be removed with pkg delete.

.. code-block:: bash

    # pkg delete packagename

Removing Packages (Example)
---------------------------

.. code-block:: bash

    # pkg delete curl
    The following packages will be deleted:

    curl-7.24.0_1

    The deletion will free 3 MB

    Proceed with deleting packages [y/N]: y
    Deleting curl-7.24.0_1... done

Upgrading Installed Packages
============================

Packages that are outdated can be found with pkg version. If a local ports tree
does not exist, pkg-version(8) will use the remote repository catalogue,
otherwise the local ports tree will be used to identify package versions.:

.. code-block:: bash

    # pkg upgrade
    Updating repository catalogue
    repo.txz        100%    297KB 296.5KB/s 296.5KB/s   00:00
    The following packages will be upgraded:

    Upgrading curl: 7.24.0 -> 7.24.0_1
    1 MB to be downloaded

    Proceed with upgrading packages [y/N]: y
    curl-7.24.0_1.txz   100% 1108KB 1.1MB/s 1.1MB/s     00:00
    Checking integrity... done
    Upgrading curl from 7.24.0 to 7.24.0_1... done

Auditing Installed Packages
===========================

Occasionally, software vulnerabilities may be discovered in software within the
Ports Collection. pkgng includes built-in auditing, similar to the
ports-mgmt/portaudit package. To audit the software installed on the system,
run:

.. code-block:: bash

    # pkg audit -F

=========================
Advanced pkgng Operations
=========================

Automatically Removing Leaf Dependencies
========================================

Removing a package may leave behind unnecessary dependencies, like
security/ca_root_nss in the example above. Such packages are still installed,
but nothing depends on them any more. Unneeded packages that were installed as
dependencies can be automatically detected and removed:

.. code-block:: bash

    # pkg autoremove
    Packages to be autoremoved:
        ca_root_nss-3.13.5

    The autoremoval will free 723 kB

    Proceed with autoremoval of packages [y/N]: y
    Deinstalling ca_root_nss-3.13.5... done

Backing Up the pkgng Package Database
=====================================

Unlike the traditional package management system, pkgng includes its own
package database backup mechanism. To manually back up the package database
contents, run::

    # pkg backup -d pkgng.db

Note: Replace the file name pkgng.db to a suitable file name.

Automatic Database Backup
=========================

Additionally, pkgng includes a periodic(8) script to automatically back up the
package database daily if daily_backup_pkgng_enable is set to YES in
periodic.conf(5).

Tip: To prevent the pkg_install periodic script from also backing up the
package database, set daily_backup_pkgdb_enable to NO in periodic.conf(5).

Restoring the pkgng Package Database
====================================

To restore the contents of a previous package database backup, run::

    # pkg backup -r pkgng.db

Removing Stale pkgng Packages
=============================

By default, pkgng stores binary packages in a cache directory as defined by
PKG_CACHEDIR in pkg.conf(5). When upgrading packages with pkg upgrade, old
versions of the upgraded packages are not automatically removed.

To remove the outdated binary packages, run:

.. code-block:: bash

    # pkg clean


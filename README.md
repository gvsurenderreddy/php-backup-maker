Author
======

 Written by Marcin Orlowski <carlos (--) wfmh (dot) org (dot) pl>

 Home: https://bitbucket.org/borszczuk/php-backup-maker/

What is it?
===========

 This is a PHP script which reads a given source directory (or multiple directories) with its subdirectories and creates set of directories filled with the content of the source directory, arranged in groups to fit a given capacity. It can be used to create CD/DVD sets for backups. It also supports ISO image creation, direct CD/DVD burning via cdrecord, automatic file index creation and more.


Features
========

 - multi-volume backups,
 - various capacity media support,
 - support for CD and DVD burning,
 - file splitting for files bigger than media capacity,
 - multi source backup,
 - on-the-fly CD/DVD burning,
 - ISO image creation,
 - optimised code to speed up ISO-over-NFS creation,
 - automatic text index creation,
 - pattern / regexp based file excluding,
 - .pdm_ignore permanent directory markers,
 - portable code,
 - bath, bed, breakfast.


Requirements
============

 - Mandatory:
    - [PHP 5.3+](http://php.net/)
 - Optional:
    - `mkisofs` if you want to create ISO images or burn CDs
    - `cdrecord` if you want to burn any CDs
    - `growisofs` if you want to burn DVDs


Configuration
=============

Most parameters can be passed as argument when script is invoked. Some of them you can default by creating php.ini settings file (this file is not required). Let's create customised pdm.ini from our template:

    cd ~
    mkdir .pdm
    cp PATH_TO_PDM_UNPACKED/pdm.ini.orig ~/.pdm/pdm.ini

Then use any text editor, and open the `pdm.ini` file, and configure whatever you wish. Detailed information is provided as comments in default ini file.

 
Configuring PHP
===============

 No special configuration is needed, but please make sure you don't have command line scripts working in Safe Mode (most probably you have this option turned off), as it needs to disable script timeout limits (it takes some time to process all the data). You also should check if memory_limit in your config file (/etc/php5/cli/php.ini) is high enough. If PHP aborts while processing your files throwing memory related errors, edit your config and increase it.


Usage
=====

Run it without arguments or with `-help` switch to get detailed usage information:

    ./pdm.php -help


How it works
============

 First, it scans source directories to learn its structure, get files and subdirs. Then it toss the data to fit in given `CD_CAPACITY` capacity. Then, depending on the work mode it either copies the data, moves it or links. Additionally it may burn it. For `iso` and `burn` it needs to create temporary data sets, which are produced as in `link` mode.


File splitting
==============

 Since release 2.6 PDM know how to handle files bigger than media size. If `-split` option is specified PDM will hunk any file that exceeds media size. However current file splitting implementation works fine, I consider it beta and due to some 'design' issues I've encountered (and wasn't able to solve in elegant manner) it won't work with `copy` and `move` modes.

 NOTE: When file is to splited, PDM will create its copy and therefore will need additional disk space. See `pdm.ini/split/buffer_size` for more information
 

Working over NFS
================

PDM does not perform any tricky actions with filesystem so be it NFS or extfs or reiserfs or anything it does not really matter. But if you use NFS, I added some switches to speed things up over networked file systems. At least for now, PDM creates bunch of symlinks in `-dest` for later processing. If your `-dest` points somewhere over NFS it means there will be additional network traffic first to create these and then to read and process. But since symlinks does not waste too much diskspace, you may wish to do the trick and separate temporary data destination directory from output directory. This works for `iso` mode only and let you specify target on local machine (i.e. `/tmp`) but at the same time point ISO destination to somewhere else (mainly over NFS). This significantly speeds things up - for usage examples see the "Examples" part below.


Examples
========

To check how many CDs you will need to backup STUFF directory:

    ./pdm.php -s=STUFF


To do the same for three diferent sources:

    ./pdm.php -s=STUFF1 -s=STUFF2 -s=STUFF3


To burn STUFF directory on the fly:

    ./pdm.php -mode=burn -s=STUFF

To create ISO images (for any reason ;-)

    ./pdm.php -m=iso -s=STUFF1 -s=STUFF2


To create ISO images over NFS *faster* than in previous release  (see "Working over NFS" for details)

    ./pdm.php -m=iso -dest=/tmp -iso-dest=/NFSMOUNT -s=STUFF

To evaluate backup size (w/o no further action) of all files but files ending with "~" (usual text editor's backup files) (Note quotes, used to avoid shell to evaluate our pattern).

    ./pdm.php -s=STUFF -ereg-pattern='.*[^~]$'

To backup your home dir but not Mozilla's cache:

    ./pdm.php -s=/home/YOU -ereg-pattern='^[.*/\.mozilla/.*/.*/Cache/.*]


I want to know more about on-the-fly burning
============================================

Bother to know more about all the CD/DVD and CD/DVD burning topics? Visit a great source of practical information at:

  http://www.cdrfaq.org/


Bugs? Suggestions?
==================

If you got any bug report or feature request. Links to right place for doing so are placed below.
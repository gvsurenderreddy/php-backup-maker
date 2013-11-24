
PDM? What a jerky name...
-------------------------------------

PDM stands for PHP Dump Maker. I wrote it, formerly as "php backup
maker", as a quick solution to my 'why-there-is-not-useful-backup-tool'
problem I've encountered one day wishing to backup some GBs of my
data to CD "my way{tm}". As it started to be useful, I decided
to give it a public release, to both help one having similar problems
I had and to give my small contribution to Open Source movement
from which I benefit now and before. The name of the tool had
to be changed from PBM as there was another project named PBM
on SourceForge.net. That's the whole story behind.


What is it?
-------------------------------------

This is a PHP script which reads a given source directory
(or multiple directories) with its subdirectories and creates
set of directories filled with the content of the source
directory, arranged in groups to fit a given capacity.
It can be used to create CD/DVD sets for backups. It also supports
ISO image creation, direct CD/DVD burning via cdrecord, automatic
file index creation and more.


Features
-------------------------------------

 - multivolume backups,
 - various capacity media support,
 - support for CD and DVD burning,
 - file splitting for files bigger than media capacity,
 - multi source backup,
 - on-the-fly CD/DVD burning,
 - ISO image creation,
 - optimised code to speed up ISO-over-NFS creation,
 - automatic text index creation,
 - pattern / regexp based file excluding,
 - .pdm_ignore permament directory markers,
 - portable code,
 - bath, bed, breakfast.


Requirements
-------------------------------------

Obligatory:

 - PHP interpreter <http://php.net/>. This script is developed
   with PHP 5.3+ in mind.

Optional:

 - mkisofs <http://freshmeat.net/projects/mkisofs/>
   if you want to create ISO images or burn CDs
 - cdrecord <http://freshmeat.net/projects/cdrecord>
   if you want to burn any CDs with this script
   (you need to configure your kernel first and config
   the script - see inside for details)
 - growisofs
   if you want to burn DVDs you would need this software


Configuration
-------------------------------------

Most parameters can be passed as argument when script
is invoked. Some of them you can default by creating
php.ini settings file (this file is not required).
Let's create customised pdm.ini from our template:

  cd ~
  mkdir .pdm
  cp PATH_TO_PDM_UNPACKED/pdm.ini.orig ~/.pdm/pdm.ini

Then use any text editor, and open the pdm.ini file,
and configure whatever you wish (pleasae note, that
the script does NOT validate these values. If you put
"abcd" as i.e. capacity you will get unexpected errors
in the script (I am not going to fix this in near future).


 General settings (PDM section):
 ----------------------------------

 media = 700

 Specify how many megabytes (MB) fits on your CDRs. 700MB
 are most widely used. Check the booklet for details.
 This can be overwriten with -media option


 ignore_file = .pdm_ignore

 This is ignore marker. Ignore marker is a file
 that tells PDM to skip the directory it resides in from
 being processed. It can be 0 bytes large (content
 just does not matter, so you can create it by calling:
    echo -n >.pdm_ignore


 ignore_subdirs = On

 If ignore_file is found PDM may also skip all subdirs
 that can be found in this directory too (this is default
 behaviour). Set it to Off, to avoid this (you'll have
 to put ignore_file in each subdir if you would like it
 to be skipped


 check_files_readability = On

 By default PDM checks if each file can be read. This
 prevents i.e. mkisofs from terminating in the middle
 of its work with error "Can't read file". However this
 option slows file processing a bit, I stronly suggest
 to keep it on for safety. If PDM is called by root,
 check_files_readability is automatically deactivated
 (as it makes no sense for might root).



 CD recording (CDRECORD section)
 -----------------------------------
 Plase note that if you don't plan to burn nor produce ISO
 images, you can ignore most of these settings. Check
 NOTEs below for more details about what to keep eye on
 mostly.


 enabled = On

 If you want to make burning available switch this to
 "On" (w/o quotes). Make sure you got external software
 like cdrecord and mkisofs installed and available.


 device = "1,0,0"

 if you want to burn anything you need to tell on which
 device your CDR burner is. Use "cdrecord -scanbus" to
 find this out, and put the 1st column value as shown
 on above example. If you don't plan to burn, nor have
 any burner, leave it as is.


 fifo_size = 10

 FIFO queue is used when you burn on-the-fly. It's memory buffer
 used to communicate between mkisofs that creates CD structures
 and cdrecord that burns your data. If it's too small, cdrecord
 may burn faster than mkisofs is able to deliver the data (this
 mostly happens if you burn thousands of millions small files
 on-the-fly). Unless you use burn-proof device, you get
 "Buffer underrun" if this happen, and in result broken CD.
 By default, 10MB is used, which shall be fine for 99,9% of
 uses. Most probably you won't ever need to touch this.




 DVD recording (GROWISOFS section)
 -----------------------------------
 Plase note that if you don't plan to burn DVDs, you can ignore
 most of these settings. Check NOTEs below for more details about
 what to keep eye on mostly.


 enabled = On

 If you want to make burning available switch this to
 "On" (w/o quotes). Make sure you got external software
 like cdrecord and mkisofs installed and available.


 device = "/dev/hdc"

 if you want to burn anything you need to tell on which
 device your DVD burner is. In contrast to cdrecord, growisofs
 uses standard device notiation.


 fifo_size = 10

 same as FIFO queue described above



 File splitting
 ------------------

 buffer_size = 0

 PDM is able to split files exceeding media capacity into
 smaller chunks to fit the media (and only such - read NOTES).
 For some reasons I decided not to use any external splitting
 tools and wrote my own function to handle that.  The only
 problem I've encountered is PHP memory management model or
 its imits to be exact. PDM deals with this as follow. If 
 buffer_size is set to 0 or is not specified, we take 3rd 
 part of chunk size or php.ini/memory_limit value, picking
 the smaller value. For example, we got 200MB file to split
 into 100MB chunks and current memory limit is set to 30MB,
 PDM will use 10MB for splitting buffer (1/3rd of 30MB).
 This slows us a bit as we have to do 10 reads/writes per
 chunk, but prevents against memory exchausting problem.
 Alternate approach is to specify buffer_size manually.
 Write how many MB you want PDM to *always* use (ie. 
 to set it up to 22MB, set buffer_size = 22). If you
 got memory_limit set high enough (i.e 200MB) 4th part
 of it is only 50MB. However PDM is memory hungry tool
 sometimes, it safely  deals with 50000 files even with
 64MB memory_limit. If you do not backup zillions of 
 files, specifing buffer_size = 100 looks quite safe
 in this example.

 NOTE: splitting is NOT used for files smaller than
       media capacity. This means PDM does NOT optimise
       media usage by using each byte of its capactity.

 If you rarely need splitting, you can leave defaults.


Configuring PHP
-------------------------------------

No special configuration is needed, but please make sure you
don't have command line scripts working in Safe Mode (most
probably you have this option turned off), as it needs to
disable script timeout limits (it takes some time to process
all the data). You also should check if memory_limit in your
config file (/etc/php4/CGI/php.ini) is high enough. I don't
belive default 8MB will do for anything. If PHP aborts while
processing your files throwing memory related errors, edit
your config and increase it. I usually got 30MB (but even
that wasn't enough for 45000 file set).


Usage
-------------------------------------

Run it without arguments or with -help switch
to get detailed usage information:

 ./pdm.php -help

Each option is described, so no need to repeat myself
here.


How it works:
-------------------------------------

First, it scans source directories to learn its structure,
get files and subdirs. Then it toss the data to fit in
given CD_CAPACITY capacity. Then, depending on the
work mode it either copies the data, moves it or links.
Additionally it may burn it.

For "iso" and "burn" it needs to create temporary data
sets, which are produced as in "link" mode.


File splitting
-------------------------------------

Since relese 2.6 PDM know how to handle files bigger than
media size. If -split option is specified PDM will hunk
any file that exceeds media size. However current file
splitting implementation works fine, I consider it beta
and due to some 'design' issues I've encountered (and
wasn't able to solve in elegant manner) it won't work with
'copy' and 'move' modes.

NOTE: When file is to splitted, PDM will create its copy
      and therefore will need additional disk space.
      Read the pdm.ini/split/buffer_size information
      above!


Working over NFS
-------------------------------------

PDM does not perform any tricky actions with filesystem
so be it NFS or ext2 or reiserfs or anything it does not
really matter. But if you use NFS, I added some switches
to speed things up over networked file systems. At least
for now, PDM creates bunch of symlinks in '-dest' for
later processing. If your '-dest' points somewhere over
NFS it means there will be additional network traffic
first to create these and then to read and process.
But since symlinks does not waste too much diskspace,
you may wish to do the trick and separate temporary data
destination directory from output directory. This works
for 'iso' mode only and let you specify target on local
machine (i.e. /tmp) but at the same time point ISO
destination to somewhere else (mainly over NFS). This
significantly speeds things up - for usage examples
see the "Examples" part below.



Examples
-------------------------------------

- to check how many CDs you will need to backup STUFF
  directory:

 ./pdm.php -s=STUFF

- to do the same for three diferent sources:

 ./pdm.php -s=STUFF1 -s=STUFF2 -s=STUFF3


- to burn STUFF directory (on the fly):

 ./pdm.php -mode=burn -s=STUFF


- to create ISO images (for any reason ;-)

 ./pdm.php -m=iso -s=STUFF1 -s=STUFF2


- to create ISO images over NFS *faster* than in previous release
  (see "Working over NFS" for details)

 ./pdm.php -m=iso -dest=/tmp -iso-dest=/NFSMOUNT -s=STUFF


- to evaluate backup size (w/o no further action) of all files
  but files ending with "~" (ususally text editor's backup files)
  (Note quotes, used to avoid shell to evaluate our pattern).

 ./pdm.php -s=STUFF -ereg-pattern='.*[^~]$'


- to backup your home dir but not Mozilla's cache:

 ./pdm.php -s=/home/YOU -ereg-pattern='^[.*/\.mozilla/.*/.*/Cache/.*]


I want to know more about CD Burning
-------------------------------------

Bother to know more about all the CDs and CD burning topics?
Visit a great source of practical information at:

  http://www.cdrfaq.org/


Bugs? Suggestions?
-------------------------------------

If you got any bug report or feature request, PLEASE, PLEASE
USE SOURCEFORGE.NET bugtracker or forum for this. DO NOT
mail me directly!


Author
-------------------------------------

 Written by Marcin Orlowski <carlos@wfmh.org.pl>
 Home page: http://freshmeat.net/projects/pdm/


--
$Id: README,v 1.20 2006/03/11 19:58:18 carl-os Exp $


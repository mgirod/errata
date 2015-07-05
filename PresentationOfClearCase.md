# Chapter 2. Presentation of ClearCase

## The main concepts

### Vobs and views

p 34

*In the second example transcript, the newline in the middle of the ls output is spurious. Correction:*

    $ ls -la /vob/myvob
    total 18
    dr-xr-xr-x 2 vobadm jgroup 1024 Oct 4 12:21 .
    drwxr-xr-x 126 root root 4096 May 5 10:53 ..
    drwxrwxr-x 3 joe jgroup 24 Aug 24 2009 dir1
    drwxrwxr-x 4 smith jgroup 46 Feb 1 2010 dir2

### Deeper into views

#### Versioning mechanism

p 36

*Typo on the first line of paragraph 4 (following the first example): missing '`/`'. Correction:*

Here the vob tag is `/vob/perl`, `/vob/perl/.@@/APP` signifies a version ...
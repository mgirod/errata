# Chapter 11. MultiSite Administration #

## Troubleshooting ##
p 277
### Insertion of new material ###
### Make Container Failed: File exists ###
The exact error, at import:
```
multitool: Error: Operation "vobsvr_make_container_permanent" failed: File exists.
```
This error is found in the technotes, in relation to an obscure race condition in 2004.
We met it however recently on v7.0.1, and found the case of interest.

The oplog being imported contained the checkin of a new version for a _text file_ element.
Looking at the (shared) source container, using the previous version, we found that the directory in the source pool was densely populated.
In fact, there were 1296 copies of the same file, with extensions running from _00_ to _zz_ (through _01_, _02_, ..., _zx_, and _zy_).

26 letters plus 10 digits, how many 2 character combinations? 36&sup;2, i.e. 1296.
The main part of the basename body was as usual a stripped _oid_, _b112fb5714c811e18f63000184985044_, but for an unknown vob object.
From the top of the contents, we found:
```
$ head -2 0-b112fb5714c811e18f63000184985044-00
^S db1 24
^V b112fb57.14c811e1.8f63.00:01:84:98:50:44 24 1 4ecb2e04
^V af803405.9f7b42d5.a41e.8d:6d:ef:e5:dd:db 24 0 4ecb2af1
```
with the synctactically restored oid: this was the version being created (version 1 on the branch), with the previous line for the previous version (0), pointing to a still existing source container in the same directory.

The explanation: the import of the new version could not be validated. It was attempted as many times as the naming scheme allowed, and failed then with a new error.

We removed the 1296 files. The next error told us about the original problem (wrong length of data file), which we solved following instructions in an [excellent tech note](http://www.ibm.com/support/docview.wss?uid=swg21221901).
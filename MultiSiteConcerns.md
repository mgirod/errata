# Chapter 5. MultiSite Concerns #

## Shortcomings of MultiSite ##
p 120
### We wrote ###
_Versioned derived objects are thus simply not visible to `lsdo` and cannot (probably) be winked in._
### Correction ###
Versioned derived objects are not visible to `lsdo`. They may however be winked in. In practice however, successful winkin on a remote site requires that _all_ the intermediate derived objects would be checked in.

Let's illustrate this by completing our example, forgetting for now the `-cr` option, hence checking in the data as well as the config record.

We checked in the `foo` derived object as version `/main/mg/3` using the view `v2` the config spec of which selected version `/main/cr/1` of the `t` directory.

This element is thus not accessible from view `v1`, which selects version `/main/mg/1` of the same t directory.
We now remove from there the original derived object, and run `clearmake`:

```
$ clearmake
Wink in derived object "foo"
$ ls -l foo
-rw-r--r-- 1 emagiro EEI-ATusers 4 Jul 7 13:14 foo
$ ct ls foo
foo@@/main/mg/3
$ ct lsdo foo
cleartool: Error: Not a derived object: "foo"
$ ct des -fmt "%m\n" foo
derived object version
$ ct dump foo | egrep '^(ref|Pre|config|Build)|cont='
source cont="/vstg/test.vbs/s/sdft/26/b/0-7bf68b49a89311e085b700156004455c-fq"
clrtxt cont="/vstg/test.vbs/c/cdft/3c/1f/7bf68b49a89311e085b700156004455c"
config rec=536984571
Predefined dependency hash=0
Build script hash=2790973829
ref count=1
```
We clearly see that while not being a derived object (hence not being connected for instance to the do-pool, and affected by scrubbing issues), this object has a reference count, a config record, and a build script.

Let us stress the point that it need not (and should not) be accessed via the config spec, except for the sole purpose of updating it), yet may be _shopped for_ and _winked in_ by `clearmake`.

There may thus be a window of opportunity for _staging_, keeping an eye on the tradeoffs of downloading large binaries.

Testing the effect of replication brings however new surprises. We skipped one step by building a compound object, i.e. one depending on an intermediate derived object, and checking in only the final one (a case thus completely disconnected from the previous one):
```
$ cat Makefile
foo: bar
      @cat bar > foo
bar:
      @echo bar > bar
$ ct catcr -r foo
----------------------------
----------------------------
Derived object: /vob/test/foo@@/main/mg/1
...
----------------------------
MVFS objects:
----------------------------
/vob/test/bar@@--07-09T12:24.17098
/vob/test/foo@@/main/mg/1
----------------------------
Build Script:
----------------------------
     @cat bar > foo
----------------------------
----------------------------
----------------------------
Derived object: /vob/test/bar@@--07-09T12:24.17098
...
----------------------------
MVFS objects:
----------------------------
/vob/test/bar@@--07-09T12:24.17098
----------------------------
Build Script:
----------------------------
      @echo bar > bar
----------------------------
$ 
```
We note first that in another view on the same site (but on a different architecture, checking this way that no dependency on local tools is creeping in), in which the foo derived object has been winked in before we made it an element, running clearmake once again gets rid of the shared derived object, and replaces it with the versioned one:
```
$ ct lsdo foo bar
--07-09T12:24  "foo@@--07-09T12:24.17099"
--07-09T12:24  "bar@@--07-09T12:24.17098"
$ clearmake
`foo' is up to date.
$ ct lsdo foo@@--07-09T12:24.17099 bar
cleartool: Error: Not a derived object: "foo@@--07-09T12:24.17099"
--07-09T12:24  "bar@@--07-09T12:24.17098"
$ ct des foo@@--07-09T12:24.17099 
cleartool: Error: Unable to access "foo@@--07-09T12:24.17099": Invalid argument.
$ ct des -s foo
foo@@/main/mg-001/1
```
Note the fact that we renamed the `mg` branch type to `mg-001` between the two transcripts, in order to hide the versioned element from the views using config specs set up to use `.../mg/LATEST` rules.

But then, on a remote site, and after waiting for the synchronization:
```
$ clearmake


$ ct lsdo foo
--07-09T18:20  "foo@@--07-09T18:20.15821"
$ ct des -fmt "%m\n" .@@/main/mg-001/1/foo/main/mg-001/1
derived object version
```
No winkin took place, despite the fact that the derived object version was indeed correctly replicated!

Removing the derived objects, and building with the verbose option shows a mention of the `bar` sub-object:
```
$ rm foo bar
$ clearmake -v
No candidate in current view for "bar"

======== Rebuilding "bar" ========
Will store derived object "/vob/test/bar"
========================================================

Must rebuild "foo" - due to rebuild of subtarget "bar"

======== Rebuilding "foo" ========
Will store derived object "/vob/test/foo"
========================================================
$ ct catcr -r .@@/main/mg-001/1/foo/main/mg-001/1
...
----------------------------
MVFS objects:
----------------------------
/vob/test/bar                 <2011-07-09T16:54:41+05:30>
/vob/test/foo@@/main/mg-001/1
----------------------------
Build Script:
----------------------------
  @cat bar > foo
----------------------------
----------------------------
----------------------------
Target bar built by mg.group
...
----------------------------
MVFS objects:
----------------------------
/vob/test/bar                 <2011-07-09T16:54:41+05:30>
----------------------------
Build Script:
----------------------------
  @echo bar > bar
----------------------------
```
Our next move will obviously be to make an element from `bar`, and to see whether this affects.

It does. But only after one explicitly removes the derived objects: only running `clearmake` leaves them as "up-to-date":
```
$ ct lsdo foo bar
--07-09T19:33  "foo@@--07-09T19:33.14947"
--07-09T19:33  "bar@@--07-09T19:33.14946"
$ clearmake
`foo' is up to date.
$ ct lsdo foo bar
--07-09T19:33  "foo@@--07-09T19:33.14947"
--07-09T19:33  "bar@@--07-09T19:33.14946"
$ rm foo bar
$ clearmake
Wink in derived object "bar"
Wink in derived object "foo"
$ ct lsdo foo bar
$ ct des -s foo bar
foo@@/main/mg-001/1
bar@@/main/mg-001/1
```
It seems to be a very tough requirement if **all** the intermediate derived objects need to be checked in!

One may consider means to support this, using `syntree`, its `-vreuse` and `-cr` options, and the _branchoff root_ strategy; let's admit that will depart from common standard procedures.

### Comment ###
We are happy that we added the word "probably", but even more to admit that our conclusion (no winkin) was incorrect.

We did not appreciate the advantages of the change we detected. Derived object versions were indeed detached from the other derived objects, but this didn't impact their being winkable, and at first sight, it did solve the issues with reference counting under MultiSite. Effectively, it makes it possible to replicate derived objects in a semi-manual way.

In retrospect, we have to take back, with an apology, the adjective naive we used to qualify all proponents of staging practices.
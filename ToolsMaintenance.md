# Chapter 8. Tools Maintenance

## Special Cases

p 190

*We wrote:*

This was an arbitrary assumption, defeated by the fact that in the context of ClearCase, the path returned would point inside the cleartext pool.

*Correction:*

Comparing the path returned under Linux to the one returned in a similar case under Solaris makes it clear that the former is incorrect.
#### RedHat 5.3 (2.6 kernel), with ClearCase 7.0.1 ####
```
$ /vob/tst/lsl -l /proc/self/exe
lrwxrwxrwx 1 shpichko admcc 0 Jul  6 11:25 /proc/self/exe -> /filer/vstg/tst.vbs/c/cdft/41/38/113d8057a7a811e08e6b0022649817a8
```
#### Solaris ####
```
$ /vob/tst/lss -l /proc/self/path/a.out
lrwxrwxrwx 1 shpichko admcc 0 Jul  6 11:19 /proc/self/path/a.out -> /vob/tst/lss
```
In the above tests, we imported to the vob the Linux version of the standard UNIX tool `ls` as `lsl`, and the Solaris version as `lss`.

We used both on hosts of the respective platforms, to show the path made available to the running process by the operating system, using the `/proc` file system (slightly different syntaxes in the two cases) and the `readlink` system call.

We can see that the path returned under Solaris is correctly virtualized, hence is usable by the process to determine colocated tools, under the same view.

This is not the case under Linux, which caused the problem we mentioned with Java 1.4.2.

*Comment:

With the original sentence, we accepted the experienced situation as a matter of fact. It did not come to our mind to compare the path returned under Linux to the one returned under Solaris.

Whereas the problem was real, hence still justifies our picking it as an example of a "special case", the root cause was a bug which IBM classified as a vendor issue, and was eventually fixed in RHEL6 and SLES11 (released in November 2010), supported by ClearCase versions 7.1.1.x and above.*
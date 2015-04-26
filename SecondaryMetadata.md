# Chapter 9. Secondary Metadata #

## p 204 ##

### We wrote ###
```
$ function ctx {
  /usr/bin/cleartool.plx $@
}
```

### Correction ###
```
$ function ctx {
  /usr/bin/cleartool.plx "$@"
}
```

### Comment ###
The double quotes preserve the integrity of shell arguments, in case they contain spaces.
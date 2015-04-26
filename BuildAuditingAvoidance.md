# Chapter 3. Build Auditing and Avoidance #

## Recording the makefiles ##
p 58

### We Wrote ###

_Like directories, they will (but only when using this special target) get recorded, but will be ignored for dependency matching purposes. Although ignored, of course as file versions and not as expanded scripts: these will still have to match as described above._

### Correction ###

Like directories, they will get recorded (but only when using this special target), but will be ignored for dependency matching purposes. The makefiles will of course be ignored only _as file versions_ and not _as expanded scripts_: the expanded scripts will still get recorded, and the records have to match, as described above.

### Comment ###

In retrospect, needlessly terse text: admittedly obscure.
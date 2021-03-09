# Implementation Details: ADIOS1/2

## Datatype Conventions

### Boolean Types

ADIOS1 (BP3) was traditionally a C library and as such as no notion for boolean types.
[At the moment](https://github.com/ornladios/ADIOS2/issues/2606), also ADIOS2 (BP4) does not define boolean types.
Although the openPMD-standard avoids using boolean types itself, developers might choose to implement them as attributes or dataset types.

#### ADIOS1

ADIOS1 is deprecated.

For compatibility, boolean types shall be written as `adios_unsigned_byte`.

On the read side, a stored boolean type is identified as `adios_unsigned_byte`, since no convention for bool-identification is specified.

#### ADIOS2

For compatibility, boolean types shall be stored as `unsigned char`.
ADIOS2 internally identifies this as `uint8_t`.

On the read side, a stored boolean *attribute* is associated with an attribute itentifying it as boolean for data readers.
The additional attribute is prefixed with `__is_boolean__/` and then adding the full path of the described attribute.

Output from `bpls -A` for a boolean attribute `pybool` stored in the location of the root group (path `/`):
```
  uint8_t         /pybool                      attr
  uint8_t         __is_boolean__/pybool        attr
```

There is no convention yet for a unique representation of ADIOS2 variables with boolean type.
Thus, implementations should cast the data to and from `unsigned char` instead.

## `stepBased` Encoding of Iterations

In order to correlate openPMD iterations with ADIOS steps, the *root* group (path `/`) in ADIOS must contain a variable:

  - `__step__`
    - type: 1-dimensional array containing N *(int)* elements, where N is the number of ADIOS steps
    - description: for each ADIOS step, this variable needs to be updated with the corresponding openPMD iteration.
    - note: ADIOS steps are absolute and not every ADIOS step or openPMD iteration contains an update for each declared openPMD record.
    - advice to implementers: [decide on this] an openPMD iteration for different openPMD records might be spread over multiple ADIOS steps.
                              An iteration of an openPMD record must correspond to exactly one ADIOS step.

## Datasets

An openPMD **data set** is represented by an group prefix that contains an ADIOS variable `__data__`.

**attributes** are defined further below and can also appear at the dataset's **group** prefix level.

## Attributes

openPMD **attributes** stored as ADIOS `Variables` at the location where they would usually be stored.

Example for a mesh record `E` with record component `x` and attributes `unitDimension` and `unitSI`:
```
  double    /data/meshes/E/unitDimension     10*{7}
  double    /data/meshes/E/x/__data__        10*{1000}
  double    /data/meshes/E/x/position        10*{1}
  double    /data/meshes/E/x/unitSI          10*scalar
```

This example uses `stepBased` iteration encoding, but other iteration encodings would work similarly with their respective `basePath` prefix.


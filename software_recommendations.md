# IAU Astrometry Data Exchange Format (ADES) Software Recommendations

This document specifies various utilities and a library API to be provided
in support of the ADES format standard.

[ADES Specification Document](http://minorplanetcenter.net/iau/info/IAU2015_ADES.pdf)


## Utilities

Command-line utilities are recommended to support interacting with
IAU astrometry data files.
Proposed utility functions are listed below organized by category:

### Format Validation

* verify file conforms to ADES XML format specification
* verify file conforms to ADES PSV format specification

### Format Conversion

* convert from XML to PSV or from PSV to XML
* convert from MPC-80-column format to XML

### File Split and Merge

* merge files
  * support both XML and PSV formats
  * allow for more than two files at a time?

* split files
  * support both XML and PSV formats
  * split by designation
  * split by observatory and/or observer?

### Other Utilities

* sort observation records by designation and date-of-observation
  * support both XML and PSV formats
  * possibly support other sort keys


------------------------------------------------------------------------------

## Library (API)

The goal of this library is to encapsulate ADES file packaging manipulation
and format specifics thereby facilitating maintenance
and future format and/or packaging changes.

Although format validation is an expensive operation on large observation files
(e.g., the entire MPC observation catalog with 10s of millions of records),
it should be enforced by default whenever reading or writing data file
content.
A switch should be provided to optionally skip content validation.

The library is implemented in C using the well supported
[libxml2](http://xmlsoft.org/index.html) library.
The core-parser of libxml2 is used to convert the specified XML
file (or stream?) into a document tree with pointers
(very efficient with respect to memory consumption)
to the various elements and parameters contained in the XML content.
Retrieval of specific records in the observation/residuals data is
accomplished by traversing this tree.

> I assume writing records to an XML file involves using libxml functions
> to initialize appropriate document/tree structures, populating those
> with data to be written, and then writing the tree to an XML file.

### Primary Functions

#### File/Stream I/O

These functions provide connections to read from and write to an
observation/residuals data file.
The read/parse function should provide a pointer tree (linked list?) to
the entire set of available records.
Similarly, only records contained in the content tree provided to the
write function will be written to the data file.

* read and parse data file
  * `ADESfilePtr = ADESopen(char *filename, ADESoptionsPtr options)`

* encode and write data file/stream
  * `ADESresultPtr = ADESwrite(char *filename, ADESdataPtr data, ADESoptionsPtr options)`

#### Content Validation

These functions provide a means of validating XML content with respect
to the specified schema.
It is expected that PSV content will be converted internally to XML
(perhaps record-by-record) and then validated against the schema
to avoid duplication of the libxml validation mechanism.

* validate the specified data file/stream
  * `ADESresultPtr = ADESvalidFile(ADESfilePtr af)`

* validate the specified header/record/element
  * `ADESresultPtr = ADESvalidHeader(ADESheadPtr header)`
  * `ADESresultPtr = ADESvalidObs(ADESobsRecPtr obsRec)`
  * `ADESresultPtr = ADESvalidRes(ADESresRecPtr resRec)`

#### Header I/O

These functions provide a means of reading and writing file header content.
Each header element will map to a key/value pair
(in the abstract sense called the "header-object").
This will be implemented using an appropriate C-language structure.

* fetch header content
  * `ADESheadPtr = ADESgetHeader(ADESfilePtr af)`

* push header content
  * `ADESresultPtr = ADESaddHeader(ADESheadPtr header, ADESheadRecPtr headRec)`

#### Observation Record I/O

These functions provide a means to read and write observation data records.
Ideally, these function will handle anything from a single observation record
to all records in the data file.
An internal pointer could be maintained allowing for a
"get the next record" capability to facilitate serial processing.

Similar to Header content, each observation record will be specified via
a set of key/value pairs.
Each record will be implemented using an appropriate c-language structure.
Multiple observation records will be represented using an array of
corresponding structures (or more likely, pointers to structures).

* fetch observation records
  * `ADESobsRecPtr = ADESgetNextObs(ADESfilePtr af, ADESoptionsPtr options)`

* push observation records
  * `ADESresultPtr = ADESaddObs(ADESdataPtr data, ADESobsRecPtr obsRec)`

#### Residual Record I/O

These functions provide a means to read and write residuals records.

> NOTE: It seems reasonable to require the `obsID` element be specified
> for each and every residual record.
> This suggests it might be nice to provide hooks in the
> "Observation Record I/O" functions to allow for corresponding residuals
> records.

* fetch residual records
  * `ADESresRecPtr = ADESgetNextRes(ADESfilePtr af, ADESoptionsPtr options)`

* push residual records
  * `ADESresultPtr = ADESaddRes(ADESdataPtr data, ADESresRecPtr resRec)`

### Convenience Functions

#### Merge

Provide a means of merging separate data content.
For example, content loaded from file A (any format) and separate
content from file B (any format) can be merged into new content and
written to a data file.

> NOTE: Are we checking for duplicate records and eliminating such?
> If so, we need to carefully define what "duplicate" means.

* merge content1 and content2 into new content
  * `ADESdataPtr = ADESmerge(ADESdataPtr data1, ADESdataPtr data2, ADESoptionsPtr options)`

#### Filter

Provide a means of extracted a subset of the available data content
matching specified constraints.
For example, select the subset matching the specified object designation
or the specified observatory code within some time period.

Although it would be nice to leave the constraint specification in an
abstract form, we will likely need to define specific allowed constraints
and perhaps limit the complexity of how multiple constraints are logically
combined.

* filter data content with specified constraints
  * `ADESdataPtr = ADESfilter(ADESdataPtr data, ADESfilterConstraintsPtr constraints, ADESoptionsPtr options)`

#### Sort

Provide a means to sort the specified content by one or perhaps multiple
keys (record fields).
For example, sort by object designation and then by date.

* sort data content using specified keys
  * `ADESdataPtr = ADESsort(ADESdataPtr data, ADESsortKeysPtr sortKeys, ADESoptionsPtr options)`


------------------------------------------------------------------------------

# Notes and Comments

## Duplicate Observation Records

In the current MPC 80-column format master observation data catalog,
apparently duplicate observations exist.
Historically, duplicate data were sometimes used to "up-weight" those data.
With current observation processing, such tricks are no longer necessary and
should not be supported.

Although the current ADES format specification does not discuss duplicate
astrometry records, it is necessary for this document to correctly define
the results of a file/data merge at the very least.

JPL defines duplicate optical records with respect to
the observation date, observatory code, R.A., declination,
and reference frame (e.g., J2000).
These fields map to `obsTime`, `std`, `ra`, `dec`,
and "J2000" (since the reference frame is implied).

This definition works reasonable well except in various pathological cases.
For example, consider an observer sending low-precision
astrometry to the MPC and high-precision astrometry to an orbit computer who
then tries to merge the low and high-precision data.


## References

Initial version of this document was partly based on input
from Michael Rudenko from an email dated 2015-Aug-14.

```
To get things rolling, here is a first attempt of a list of some of the things
I think we'll need for the base utilities and low-level C library.
Changes and/or additional items welcome.

Stand alone utilities:

1. Convert MPC 1992 80-column format to XML.

2a. Convert XML files to PSV.
2b. Convert PSV files to XML.

3a. Strip headers from XML files.
3b. Strip headers from PSV files.

4a. Merge XML files.
4b. Merge PSV files.

5a. Spilt XML files by designation.
5b. Split PSV files by designation.

6a. Sort XML files (without headers) by designation and date-obs.
6b. Sort PSV files (without headers) by designation and date-obs.

7a. Validate XML file format.
7b. Validate PSV file format.


Library functions:

1. Fetch headers.
2. Fetch observation blocks.
3. Fetch all observations.
4. Fetch a list of designations.
5. Fetch all observations for a given designation.
6. Fetch a list of observatory codes.
7. Fetch all observations for a given observatory code.
8. Convert from date-obs to Julian date.
9. Convert from sexigesimal to degrees.
10. Convert from degrees to sexigesimal.
```


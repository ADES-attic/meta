### Some thoughts

The Libxml2 C library looks like the starting point for everything.  I've never used it.  It'll be interesting.

Prelimary exercises:

1.  Go through the relevant provided examples..
2.  Modify examples to read and validate an ADES file.

Then probably there are several possible directions, some of which can be explored in parallel.

### XSLT XML->PSV project
1.  Explore libxslt, there is a provided tutorial.
2.  At a glance, it looks like libxslt works on the data structure built by libxml2.
3.  Learn some XSLT, it should be possible to define style sheets to generate XML and PSV.
4.  A program using libxml2, libxslt, and a PSV style sheet should be able to do XML to PSV file conversion.  The program should be short, just calling the library functions.  It should do the same thing you could do just as well with the provided command line utility xsltproc.

### PSV->libxml2 project
1.  There are provided examples of ad-hoc construction of an in-memory tree using the data structures of libxml2/libxslt.
2.  The examples might be modified to read PSV and construct the tree.
 
### Direct PSV->XML project.
1.  libxml2 has functions for writing an in-memory tree to serialized XML.  Given the completed PSV->libxml2 project, see if it isn't trivial to complete PSV to XML file conversion.

### Representation project
1.  The doc tree of libxml2 allows for the full the generality of XML.  I think we are likely to want to define data structures specialized just for ADES.
2.  The project would be to look at the data structures of libxml2 and define a new ADES data structure with non-libxml2 components.  Goals would be 1) to make the structure as simple as possible and 2) to allow converion to and from the libxml2 tree to be as simple and efficient as possible.
3.  Proof of concept would be a function to convert a libxml2 tree to an ADES data structure and a demo program that reads an XML file into the libxml2 tree, converts to the ADES data structure, and dumps the data struture to show that it works.


### PSV->ADES struct project
1.  Given data structures defined with the represenation project, write a reader that reads from PSV directly into these structures, no lib-anything required.
2.  Parsing code from the PSV->libxml2 project may be useful.  Steal code where appropriate.
2.  Demo program outputs result using dump code from representation project.

### ADES struct to libxml2 tree project.
1.  Write a conversion function.
2.  Demo with code from representation project showing XML->libxml2 tree->ADES struct (destroy libxml2 tree) ADES struct->libxml2 tree->XML
3.  Also demo with code from PSV->ADES struct project showing PSV->ADES struct->libxml2 tree->XML.

### Evaluate PSV->XML techniques
Direct PSV->XML and PSV->ADES struct->XML represent two ways to do file conversion.  Direct may have fewer lines of code and be more efficient but the decomposition into the two steps of going to and from the ADES struct may leave those steps individually smaller and more maintainable.  Further, if the ADES struct proves useful for library functions, the steps might have separate uses.

### 24 Sep 2015, Evaluation time
So I implemented the two ways to do PSV to XML.

Direct PSV->XML looked slick at first, but the XML output retained whatever element order was in the PSV and didn't follow the ADES standard order.  A tricky hack fixed the field order in observation records but it's not ideal.  It's tricky, it uses a little temporary workspace, and it only fixes field order, not header order.

Two step library-based conversion uses lots of cut and paste code.  LOC is several times that of the direct converter, and it doesn't even validate yet.

Neither technique do streaming.

Refactor:

* pt, let's call it, would be a function that parses PSV to a libxml2 tree, annotating with line numbers.
* similarly the standard xml read function parses to a tree annotated with line numbers.
* validation on an annotated tree gives good error messages.  (validating xml parse does it in one step)
* the good way to get the ADES standard output order is to populate the C-structs.  (the tree doesn't do it, the pxdirect technique is a too-clever partial hack.)  Libxml2 tree to C-struct would be an internal non-API function. Let's call it tc.
* API function readPSV then would call pt, validate if requested, and call tc if requested.
* API function readXML would read, validating if requested, and call tc if requested.
* API function writePSV goes from C-struct to PSV. This is straightforward in C.  It might also be possible with XSLT.  There is no validation involved here.  If final validation is wanted, it must be done on the output file as a separate step.
* API function writeXML goes from C-struct to XML through a temporary libxml2 tree. This also is non-validating.  While it is tempting to validate the temporary tree, error messages will be unhelpful without line numbers.  If final validation is wanted, it must be done on the output file as a separate step.

These four API functions should sufficient for in-place validation of both file times and conversion both ways, with or without validation.

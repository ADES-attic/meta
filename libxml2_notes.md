A few notes for installing and using libxml2

(skeys, Aug 2015)

I tried installing from a git clone but ran into a problem generating the
`configure` script.  Installing from a tarball went better; the tarball has
a premade configure script.

On our system at the MPC, there distro has an old libxml2 in `/usr`.
By default the configure installs to `/usr/local`, so that works well.

The example command to build the example files worked well, propererly
recognizing the /usr/local include and lib paths.

When running a program built this way though, the Linux program loader must
know to use `/usr/local/lib` for loading libraries at load time, and our
default shell does not have this path.  The solution is `LD_LIBRARY_PATH`
either specified on the bash command line as in

    LD_LIBRARY_PATH=/usr/local/lib ./tree2

or more conviently, by exporting the symbol from a `.bash_profile` or similar.

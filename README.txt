Each ROM here has its own separate README file that describes where there
is.

To avoid code duplication there may be dependencies between them in build
processes.

For example, RAM_Manager depends on files from HostFS for the embedded
HostFS and UPURSFS routines.

For that reason it is recommended that anyone who clones or forks this
repo does it at the top level.  It may end up checking out unnecessary
code, but it's a small repo!

Pull requests and refactoring ideas welcome!

In general the Makefiles have been created to work on Linux machines
with "beebasm" on the PATH.

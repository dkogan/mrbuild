# -*- Makefile -*-

PYTHON_VERSION_FOR_EXTENSIONS := 3
include /usr/include/mrbuild/Makefile.common.header

# This is a sample Makefile using the Makefile.common infrastructure. A quick
# way to bootstrap a new project is to copy this file to the root directory of
# the project and then to modify each variable to fit that particular project.

# The name of the project. By convention, libraries should be called lib... but
# this isn't required
PROJECT_NAME := libfrobnicator

# The version of the library. We treat the major version as the version of the
# ABI/API. So every time we change the ABI or an API in a backwards-incompatible
# way, we bump the ABI_VERSION. If we make non-breaking changes, bumping the
# TAIL_VERSION is sufficient. In this example, the full version is 0.1
ABI_VERSION  := 0
TAIL_VERSION := 1

# Build all C and C++ sources in src/ into the library
LIB_SOURCES := src/*.c*

# Build all C and C++ sources in bin/ into separate executables
BIN_SOURCES := bin/*.c*

# If bin/run_foo.c exists, it is picked up in BIN_SOURCES, and the bin/run_foo
# executable will be built from the library and bin/run_foo.o (built from
# bin/run_foo.c). This is the default behavior and nothing needs to be specified

# I specify that bin/run_foo2 consists of the library and bin/run_foo2.o (as
# usual) AND links with bin/run_foo2_extra.o. The latter will be built from
# bin/run_foo2_extra.c (or .cc or .cpp and so on, whichever exists)
bin/run_foo2: bin/run_foo2_extra.o

# Suppose I have bin/run_foo3.c to build bin/run_foo3. And suppose bin/run_foo3
# needs to additionally build with sources generated from run_foo3.in: the .o
# links with bin/run_foo3_generated.o (built from bin/run_foo3_generated.c) and
# the .c #includes run_foo3_generated.h, and that both of these are generated
# from run_foo3.in. We specify this in the usual way, with a tiny bit of
# mrbuild-specific stuff:
run_foo3:   run_foo3_generated.o
run_foo3.o: run_foo3_generated.h
%3_generated.h %3_generated.c: %3.in
	make_generated_files $<
EXTRA_CLEAN += run_foo3_generated.h run_foo3_generated.c
# If we're using gengetopt to generate the sources, the build rule and the
# EXTRA_CLEAN list above are provided in mrbuild, and can be omitted.

# Any extra flags to pass to the C and C++ compilers. The build system always
# builds with debugging information and (if no other -O flag is given) with
# optimization. Use += to not override any settings from the commandline
CCXXFLAGS += -Wno-unused-parameter

# Extra flags to pass to the C compiler when building src/bleh.o from src/bleh.c
src/bleh.o: CFLAGS += -DFOO

# Link bin/run_foo with -lbar. Do NOT link the library with -lbar.
bin/run_foo: LDLIBS += -lbar

# Link all the executables AND the library with -lzap
LDLIBS += -lzap

# If we have doxygen docs, we can state the rule to build them. Everything will
# be built into doxygen-doc/, the DIST_DOC and DIST_MAN distribution lists below
# install the man-pages and the html docs
doc: doxygen-doc/
doxygen-doc/: frobnicator.dox
	SRCDIR='.' PROJECT='frobnicator' DOCDIR=$@ VERSION='$(VERSION)' PERL_PATH='/bin/perl' HAVE_DOT='YES' DOT_PATH='/bin' GENERATE_MAN='YES' GENERATE_RTF='NO' GENERATE_XML='NO' GENERATE_HTMLHELP='NO' GENERATE_CHI='NO' GENERATE_HTML='YES' GENERATE_LATEX='NO' doxygen $<
doxygen-doc/%: doxygen-doc/ ;
.PHONY: doc
EXTRA_CLEAN += doxygen-doc

# distribute (in a 'make install' and thus into the package) all headers in src/
# except those that match src/*tst*
DIST_INCLUDE        := src/*.h
DIST_INCLUDE_EXCEPT := src/*tst*

# distribute (in a 'make install' and thus into the package) all executables
# built from BIN_SOURCES except bin/*_tst. And ship the python application
DIST_BIN := $(filter-out bin/%_tst,$(wildcard $(BIN_TARGETS))) python-tool

# distribute all generated manpages in section 3 EXCEPT those for the test
# program
DIST_MAN                 := doxygen-doc/man/man3
DIST_MAN_EXCEPT_FINDSPEC := -type f -name '*_tst.3'

# distribute the html documentation
DIST_DOC := doxygen-doc/html


# This is the manpage-generating technique from
# http://notes.secretsauce.net/notes/2018/10/23_manpages-and-readmes.html
#
# generate manpages from distributed binaries, and ship them.
DIST_MAN += $(addsuffix .1,$(DIST_BIN))
$(DIST_MAN): %.1: %.pod
	pod2man --center="title: does something" --name=THING --release="thing 0.1" --section=1 $< $@
%.pod: %
	make-pod-from-help $< > $@
	cat footer.pod >> $@
EXTRA_CLEAN += $(DIST_MAN) $(patsubst %.1,%.pod,$(DIST_MAN))

# This is the python-extension-generating technique from
# http://notes.secretsauce.net/notes/2017/11/14_python-extension-modules-without-setuptools-or-distutils.html
frobnicator_pywrap.o: CFLAGS += $(PY_MRBUILD_CFLAGS)
frobnicator_pywrap.o: $(addsuffix .h,$(wildcard *.docstring))
frobnicator/_frobnicator$(PY_EXT_SUFFIX): frobnicator_pywrap.o libfrobnicator.so
	$(PY_MRBUILD_LINKER) $(PY_MRBUILD_LDFLAGS) $< -lfrobnicator -o $@
# The python libraries (compiled ones and ones written in python) all live in
# frobnicator/
DIST_PY3_MODULES := frobnicator
all: frobnicator/_frobnicator$(PY_EXT_SUFFIX)
EXTRA_CLEAN += frobnicator/*.so


include /usr/include/mrbuild/Makefile.common.footer

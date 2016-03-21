all:

SHELL       := /bin/sh

# Support out-of-tree builds
srcdir      := .
VPATH       := .

prefix      := /usr/local
exec_prefix := ${prefix}
datarootdir := ${prefix}/share
bindir      := ${exec_prefix}/bin
datadir     := ${datarootdir}
sysconfdir  := ${prefix}/etc
libdir      := ${exec_prefix}/lib
sbindir     := ${exec_prefix}/sbin
localstatedir := ${prefix}/var
CXX      := g++
# CXXFLAGS are for the main binary, so don't use them here, use MODFLAGS instead
MODFLAGS := -I$(srcdir)/../include -I../include    -std=c++11 -O2 -D_FORTIFY_SOURCE=2 -Wall -W -Wno-unused-parameter -Woverloaded-virtual -Wshadow -pthread   -fvisibility=hidden -fPIC -include znc/zncconfig.h
MODLINK  := -shared
LDFLAGS  := -Wl,--export-dynamic
ISCYGWIN := 

# LIBS are not and should not be used in here.
# The znc binary links already against those.
# ...but not on cygwin!
LIBS     :=
ifeq "$(ISCYGWIN)" "1"
LIBS     += -ldl  -pthread -lssl -lcrypto  -lz -licuuc -licudata 
endif

PERL_ON  := no
PERL     := 
PYTHON_ON:= no
PY_CFLAGS:= 
PY_LDFLAGS:=
SWIG     := 
MODDIR   := ${exec_prefix}/lib/znc
DATADIR  := ${datarootdir}/znc
LIBZNC   := 
LIBZNCDIR:= 
INSTALL         := /usr/bin/install -c
INSTALL_PROGRAM := ${INSTALL}
INSTALL_SCRIPT  := ${INSTALL}
INSTALL_DATA    := ${INSTALL} -m 644
SED := /usr/bin/sed

TCL_FLAGS:= 

ifneq "$(V)" ""
VERBOSE=1
endif
ifeq "$(VERBOSE)" ""
Q=@
E=@echo
C=-s
else
Q=
E=@\#
C=
endif

ifneq "$(LIBZNC)" ""
LIBS += -L.. -lznc -Wl,-rpath,$(LIBZNCDIR)
endif

CLEAN    :=

FILES    := $(notdir $(wildcard $(srcdir)/*.cpp))

#include $(srcdir)/modperl/Makefile.inc
#include $(srcdir)/modpython/Makefile.inc
#include $(srcdir)/modtcl/Makefile.inc

FILES    := $(basename $(FILES))

ifeq "" "1"
FILES    := $(foreach file, $(FILES), \
	$(if $(shell grep REQUIRESSL $(srcdir)/$(file).cpp), \
		, \
		$(basename $(file)) \
	))
endif

ifeq "" ""
FILES    := $(shell echo $(FILES) | sed -e "s:cyrusauth::")
endif
cyrusauthLDFLAGS := -lsasl2

TARGETS  := $(addsuffix .so, $(FILES))

CLEAN    += *.so *.o

.PHONY: all clean install install_datadir uninstall
.SECONDARY:

all: $(TARGETS)

install: all install_datadir
	$(INSTALL_PROGRAM) $(TARGETS) $(DESTDIR)$(MODDIR)

install_datadir:
	rm -rf $(DESTDIR)$(DATADIR)/modules
	test -d $(DESTDIR)$(MODDIR) || $(INSTALL) -d $(DESTDIR)$(MODDIR)
	test -d $(DESTDIR)$(DATADIR)/modules || $(INSTALL) -d $(DESTDIR)$(DATADIR)/modules
	rm -rf $(DESTDIR)$(MODDIR)/*.so
	cp -R $(srcdir)/data/* $(DESTDIR)$(DATADIR)/modules
	find $(DESTDIR)$(DATADIR)/modules -type d -exec chmod 0755 '{}' \;
	find $(DESTDIR)$(DATADIR)/modules -type f -exec chmod 0644 '{}' \;

clean:
	rm -rf $(CLEAN)

%.o: %.cpp Makefile
	@mkdir -p .depend
	$(E) Building module $(notdir $(basename $@))...
	$(Q)$(CXX) $(MODFLAGS) -c -o $@ $< $($(notdir $(basename $@))CXXFLAGS) -MD -MF .depend/$(notdir $@).dep

%.so: %.o Makefile
	$(E) "Linking  module" $(notdir $(basename $@))...
	$(Q)$(CXX) $(MODFLAGS) $(LDFLAGS) $(MODLINK) -o $@ $< $($(notdir $(basename $@))LDFLAGS) $(LIBS)

uninstall:
	# Yes, we are lazy, just remove everything in there
	rm -rf $(DESTDIR)$(MODDIR)/*
	rm -rf $(DESTDIR)$(DATADIR)/*
	rmdir $(DESTDIR)$(MODDIR)
	rmdir $(DESTDIR)$(DATADIR)

-include $(wildcard .depend/*.dep)

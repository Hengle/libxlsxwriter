###############################################################################
#
# Makefile for libxlsxwriter library.
#
# Copyright 2014-2021, John McNamara, jmcnamara@cpan.org
#

# Keep the output quiet by default.
Q=@
ifdef V
Q=
endif

DESTDIR ?=
PREFIX  ?= /usr/local
OPT_LEVEL ?= -O3

# Directory variables.
OBJS_DIR    = .
INC_DIR     = ../include

LIBS = -lz

ifdef USE_SYSTEM_MINIZIP
LIBS   += -lminizip
CFLAGS += -DUSE_SYSTEM_MINIZIP
else
# Use the local copy of minizip.
MINIZIP_DIR = ../third_party/minizip
MINIZIP_OBJ = $(MINIZIP_DIR)/ioapi.o  $(MINIZIP_DIR)/zip.o
MINIZIP_SO  = $(MINIZIP_DIR)/ioapi.so $(MINIZIP_DIR)/zip.so
endif

ifdef USE_STANDARD_TMPFILE
# Use standard/C tmpfile().
CFLAGS += -DUSE_STANDARD_TMPFILE
else
# Use tmpfileplus (the default).
TMPFILEPLUS_DIR = ../third_party/tmpfileplus
TMPFILEPLUS_OBJ = $(TMPFILEPLUS_DIR)/tmpfileplus.o
TMPFILEPLUS_SO  = $(TMPFILEPLUS_DIR)/tmpfileplus.so
endif

ifdef USE_NO_MD5
# Don't use MD5 to avoid duplicate image files.
CFLAGS += -DUSE_NO_MD5
else
# Use md5 (the default).
MD5_DIR = ../third_party/md5
MD5_OBJ = $(MD5_DIR)/md5.o
MD5_SO  = $(MD5_DIR)/md5.so
endif

# Set flag for big endian architecture.
ifdef USE_BIG_ENDIAN
CFLAGS += -DLXW_BIG_ENDIAN
endif

# Use a user-defined double number formatting function.
ifdef USE_DOUBLE_FUNCTION
CFLAGS += -DUSE_DOUBLE_FUNCTION
endif

# Use fmemopen() to avoid creating certain temporary files
ifdef USE_FMEMOPEN
CFLAGS += -DUSE_FMEMOPEN
endif

# Flags passed to compiler.
CFLAGS   += -g $(OPT_LEVEL) -Wall -Wextra -Wstrict-prototypes -pedantic -ansi

# Fix for modified zconf.h on Gentoo.
ifneq (,$(findstring gentoo, $(shell uname -sr)))
CFLAGS += -DOF=_Z_OF
endif


# Ignore icc remarks about inlining limits.
ifeq ($(CC),icc)
CFLAGS   +=-diag-disable=11074,11076
endif

# Library names.
LIBXLSXWRITER_A  = libxlsxwriter.a
LIBXLSXWRITER_SO = libxlsxwriter.so

# Library with additional non-static functions for testing.
LIBXLSXWRITER_TO = libxlsxwriter_test.a

# Flags passed to static linker.
ARFLAGS = rc

# Flags passed to dynamic linker.
FPIC = -fPIC
SOFLAGS = -shared $(FPIC)

# Get Env/OS name.
UNAME := $(shell uname)

# Change make options on OS X.
ifeq ($(UNAME), Darwin)
LIBXLSXWRITER_SO = libxlsxwriter.dylib
SOFLAGS = -dynamiclib $(FPIC) -install_name $(PREFIX)/lib/$(LIBXLSXWRITER_SO)
endif

# Check for MinGW/MinGW64/Cygwin environments.
ifneq (,$(findstring MINGW, $(UNAME)))
MING_LIKE = y
endif
ifneq (,$(findstring MSYS, $(UNAME)))
MING_LIKE = y
endif
ifneq (,$(findstring CYGWIN, $(UNAME)))
MING_LIKE = y
endif

# Change make options on MinGW/MinGW64/Cygwin.
ifdef MING_LIKE
LIBXLSXWRITER_SO = libxlsxwriter.dll
FPIC =
CC = gcc
CFLAGS += -Wno-char-subscripts -Wno-long-long
endif

# Headers as dependecies.
HDRS = $(wildcard ../include/xlsxwriter/*.h)

# Ojects to compile.
SRCS  = $(wildcard *.c)
OBJS  = $(patsubst %.c,%.o,$(SRCS))
SOBJS = $(patsubst %.c,%.so,$(SRCS))
TOBJS = $(patsubst %.c,%.to,$(SRCS))
# End of OBJS

# Build the object files and the libraries.
all : $(LIBXLSXWRITER_A) $(LIBXLSXWRITER_SO)
	$(Q)cp $(LIBXLSXWRITER_A) $(LIBXLSXWRITER_SO) ../lib

test_lib : libxlsxwriter_test.a

# The static library.
$(LIBXLSXWRITER_A) : $(OBJS)
	$(Q)$(AR) $(ARFLAGS) $@ $(MINIZIP_OBJ) $(TMPFILEPLUS_OBJ) $(MD5_OBJ) $^

# The dynamic library.
ifeq ($(findstring m32,$(CFLAGS)),m32)
ARCH = -m32
endif

$(LIBXLSXWRITER_SO) : $(SOBJS)
	$(Q)$(CC) $(LDFLAGS) $(SOFLAGS) $(ARCH) $(TARGET_ARCH) -o $@ $(MINIZIP_SO) $(TMPFILEPLUS_SO) $(MD5_SO) $^ $(LIBS)

# The test library.
$(LIBXLSXWRITER_TO) : $(TOBJS)
	$(Q)$(AR) $(ARFLAGS) $@ $(MINIZIP_OBJ) $(TMPFILEPLUS_OBJ) $(MD5_OBJ) $^

# Minimal target for quick compile without creating the libs.
test_compile : $(OBJS)

# Targets for the object files.
%.o  : %.c $(HDRS)
	$(Q)$(CC) -I$(INC_DIR) $(CFLAGS) $(CXXFLAGS) $(TARGET_ARCH) -c $<

%.so : %.c $(HDRS)
	$(Q)$(CC) $(FPIC) -I$(INC_DIR) $(LDFLAGS) $(CFLAGS) $(CXXFLAGS) $(TARGET_ARCH) -c $< -o $@

%.to : %.c $(HDRS)
	$(Q)$(CC) -g -O3 -DTESTING -I$(INC_DIR) $(CFLAGS) $(CXXFLAGS) -c $< -o $@


# Clean up any temp/build files.
clean :
	$(Q)rm -f *.o *.a *.so *.to *.dylib *.dll

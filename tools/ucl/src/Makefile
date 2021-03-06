#             __________               __   ___.
#   Open      \______   \ ____   ____ |  | _\_ |__   _______  ___
#   Source     |       _//  _ \_/ ___\|  |/ /| __ \ /  _ \  \/  /
#   Jukebox    |    |   (  <_> )  \___|    < | \_\ (  <_> > <  <
#   Firmware   |____|_  /\____/ \___  >__|_ \|___  /\____/__/\_ \
#                     \/            \/     \/    \/            \/
# $Id$
#
CFLAGS += -I../include

ifndef V
SILENT = @
endif

ifeq ($(OS),Windows_NT)
mkdir = if not exist $(subst /,\,$(1)) mkdir $(subst /,\,$(1))
else
mkdir = mkdir -p $(1)
endif

ifdef RBARCH
CFLAGS += -arch $(RBARCH)
endif

CPPDEFINES := $(shell echo foo | $(CROSS)$(CC) -dM -E -)
#build standalone win32 executables on cygwin
ifeq ($(findstring CYGWIN,$(CPPDEFINES)),CYGWIN)
COMPILETARGET = cygwin
else
ifeq ($(findstring MINGW,$(CPPDEFINES)),MINGW)
COMPILETARGET = mingw
else
# OS X specifics. Needs to consider cross compiling for Windows.
ifeq ($(findstring APPLE,$(CPPDEFINES)),APPLE)
# When building for 10.4+ we need to use gcc. Otherwise clang is used, so use
# that to determine if we need to set arch and isysroot.
ifeq ($(findstring __clang__,$(CPPDEFINES)),__clang__)
CFLAGS += -mmacosx-version-min=10.5
else
# when building libs for OS X 10.4+ build for both i386 and ppc at the same time.
# This creates fat objects, and ar can only create the archive but not operate
# on it. As a result the ar call must NOT use the u (update) flag.
ARCHFLAGS += -arch ppc -arch i386
# building against SDK 10.4 is not compatible with gcc-4.2 (default on newer Xcode)
# might need adjustment for older Xcode.
CC = gcc-4.0
CFLAGS += -isysroot /Developer/SDKs/MacOSX10.4u.sdk -mmacosx-version-min=10.4
NATIVECC ?= gcc-4.0
endif
COMPILETARGET = darwin
else
COMPILETARGET = posix
endif
endif
endif
$(info Compiler creates $(COMPILETARGET) binaries)

TARGET_DIR ?= $(shell pwd)/
OBJDIR = $(TARGET_DIR)build$(RBARCH)

SOURCES = alloc.c io.c n2b_99.c n2b_d.c n2b_ds.c n2b_to.c n2d_99.c	\
	n2d_d.c n2d_ds.c n2d_to.c n2e_99.c n2e_d.c n2e_ds.c n2e_to.c ucl_crc.c	\
	ucl_init.c ucl_ptr.c ucl_str.c ucl_util.c #ucl_dll.c

OBJS =  $(addprefix $(OBJDIR)/,$(SOURCES:%.c=%.o))

# This Makefile is _not_ used to build uclpack for Rockbox builds anymore (they
# use tools.make), so we can use $(CC) and $(AR) here.

libucl$(RBARCH).a: $(TARGET_DIR)libucl$(RBARCH).a

dll: ucl.dll
ucl.dll: $(TARGET_DIR)ucl.dll
$(TARGET_DIR)ucl.dll: $(OBJS)
	@echo DLL $(notdir $@)
	$(SILENT)$(CROSS)$(CC) $(CFLAGS) -shared -o $@ $^ \
		    -Wl,--output-def,$(TARGET_DIR)ucl.def

$(TARGET_DIR)libucl$(RBARCH).a: $(OBJS)
	@echo AR $(notdir $@)
	$(SILENT)$(CROSS)$(AR) rcs $@ $(OBJS)

$(OBJDIR)/%.o: %.c
	@echo CC $<
	$(SILENT)$(call mkdir, $(dir $@))
	$(SILENT)$(CROSS)$(CC) $(CFLAGS) $(ARCHFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET_DIR)libucl*.a
	rm -rf build*


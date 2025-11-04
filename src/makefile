#-----------------------------
# Zzip/Zzlib Makefile
#-----------------------------

CC			= gcc
LD			= gcc
STRIP		= strip

WARNING		= -W -Wall -Wshadow -Wpointer-arith -Winline
# Optionnal:
# -fprofile-arcs -fbranch-probabilities
OPTFLAGS	= -O3 -ffast-math -fforce-addr -fstrict-aliasing -fomit-frame-pointer -foptimize-sibling-calls

# Detect OS and arch automatically
UNAME_S := $(shell uname -s 2>/dev/null || echo Unknown)
UNAME_M := $(shell uname -m 2>/dev/null || echo Unknown)

# Windows detection (MSYS/Cygwin/MinGW variants)
ifneq (,$(findstring MINGW,$(UNAME_S)))
	OS_IS_WINDOWS := 1
endif
ifneq (,$(findstring MSYS,$(UNAME_S)))
	OS_IS_WINDOWS := 1
endif
ifneq (,$(findstring CYGWIN,$(UNAME_S)))
	OS_IS_WINDOWS := 1
endif

ifeq ($(OS_IS_WINDOWS),1)
	OSFLAGS := -DWIN32
else
	OSFLAGS := -DUNIX
endif

# Per-platform extensions and link flags
ifeq ($(OS_IS_WINDOWS),1)
	# Windows (MinGW/MSYS/Cygwin) style
	EXEEXT := .exe
	DLL_EXT := .dll
	# Use --out-implib on Windows to produce an import lib and --kill-at for stdcall name decorations
	DLL_OUT_FLAGS := -shared -Wl,--out-implib,libzzlib.a,--kill-at
	PICFLAGS :=
else
	# Unix/Linux style
	EXEEXT :=
	DLL_EXT := .so
	# Set SONAME on linux
	DLL_OUT_FLAGS := -shared -Wl,-soname,libzzlib.so
	# Compile position-independent code for shared libraries
	PICFLAGS := -fPIC
endif

# CPU arch flags (x86_64 vs x86)
ifeq ($(UNAME_M),x86_64)
	ARCH_FLAGS := -march=x86-64 -mtune=generic -malign-double -m64
else
	# default to 32-bit x86 optimizations; if host is not x86, leave empty
	ifneq (,$(findstring i386,$(UNAME_M)))
		ARCH_FLAGS := -march=pentium4 -mtune=pentium4 -malign-double -m32
	else
		ARCH_FLAGS :=
	endif
endif

# C standard to use with GCC/Clang
STD := -std=gnu99

CFLAGS		= $(ARCH_FLAGS) $(OPTFLAGS) $(OSFLAGS) $(WARNING) $(PICFLAGS) -c $(STD)
LFLAGS		= $(ARCH_FLAGS)

OBJS		= coding.o struct_model0.o struct_model1.o
OBJS_SFX	= zzipstub.o coding-sfx.o struct_model0-sfx.o struct_model1-sfx.o block-sfx.o
OBJS_ZZIP	= $(OBJS) zzip.o block.o bwt.o
OBJS_ZZLIB	= $(OBJS) bwt-dll.o block-dll.o

ZZIP_EXE	= zzip$(EXEEXT)
MAKE_H_EXE	= make_h$(EXEEXT)
SFX_EXE		= zzip-sfx$(EXEEXT)
ZZLIB_DLL	= zzlib$(DLL_EXT)
ZZLIB_A		= libzz.a

all : $(ZZIP_EXE)

sfx : $(SFX_EXE)

dll : $(ZZLIB_DLL)

clean :
	rm -f *.o zzipstub.sfx sfx_code.h $(ZZIP_EXE) $(MAKE_H_EXE) $(SFX_EXE) $(ZZLIB_DLL) libzzlib.a $(ZZLIB_A)

$(MAKE_H_EXE) : make_h.c
	$(LD) $(LFLAGS) -o $(MAKE_H_EXE) make_h.c
	$(STRIP) $(MAKE_H_EXE)

sfx_code.h : $(MAKE_H_EXE)
	$(MAKE_H_EXE) zzipstub.sfx > sfx_code.h

$(SFX_EXE) : zzip-sfx.c global.h zzipstub.sfx
	$(LD) $(LFLAGS) -o $(SFX_EXE) zzip-sfx.c
	$(STRIP) $(SFX_EXE)

$(ZZIP_EXE) : $(OBJS_ZZIP)
	$(LD) $(LFLAGS) -o $(ZZIP_EXE) $(OBJS_ZZIP)
	$(STRIP) $(ZZIP_EXE)

zzipstub.sfx : $(OBJS_SFX)
	$(LD) $(LFLAGS) -o zzipstub.sfx $(OBJS_SFX)
	$(STRIP) zzipstub.sfx
	$(MAKE_H_EXE) zzipstub.sfx > sfx_code.h
	$(CC) $(CFLAGS) -DSFX -Os -o block-sfx.o block.c
	$(LD) $(LFLAGS) -o zzipstub.sfx $(OBJS_SFX)
	$(STRIP) zzipstub.sfx
	$(MAKE_H_EXE) zzipstub.sfx > sfx_code.h

$(ZZLIB_DLL) : $(OBJS_ZZLIB)
	$(LD) $(LFLAGS) $(DLL_OUT_FLAGS) -o $(ZZLIB_DLL) $(OBJS_ZZLIB)
	$(STRIP) $(ZZLIB_DLL)

lib : $(OBJS_ZZLIB)
	ar rcs $(ZZLIB_A) $(OBJS_ZZLIB)
	
$(OBJS_SFX) $(OBJS_ZZLIB) $(OBJS_ZZIP) : global.h zzip.h

zzip.o : zzip.c
	$(CC) $(CFLAGS) -DZZIP zzip.c

block.o : block.c
	$(CC) $(CFLAGS) -DZZIP block.c

bwt.o : bwt.c
	$(CC) $(CFLAGS) -DZZIP bwt.c

coding.o : coding.c
	$(CC) $(CFLAGS) -funroll-loops coding.c

struct_model0.o : struct_model0.c ac-common.h
	$(CC) $(CFLAGS) struct_model0.c

struct_model1.o : struct_model1.c ac-common.h
	$(CC) $(CFLAGS) struct_model1.c

block-dll.o : block.c
	$(CC) $(CFLAGS) -DZZLIB -o block-dll.o block.c

bwt-dll.o : bwt.c
	$(CC) $(CFLAGS) -DZZLIB -o bwt-dll.o bwt.c

zzipstub.o : zzip.c
	$(CC) $(CFLAGS) -DSFX -Os -o zzipstub.o zzip.c

block-sfx.o : block.c sfx_code.h
	$(CC) $(CFLAGS) -DSFX -Os -o block-sfx.o block.c

coding-sfx.o : coding.c
	$(CC) $(CFLAGS) -DSFX -o coding-sfx.o coding.c

struct_model0-sfx.o : struct_model0.c ac-common.h
	$(CC) $(CFLAGS) -DSFX -o struct_model0-sfx.o struct_model0.c

struct_model1-sfx.o : struct_model1.c ac-common.h
	$(CC) $(CFLAGS) -DSFX -o struct_model1-sfx.o struct_model1.c

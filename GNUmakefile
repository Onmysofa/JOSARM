OBJDIR := obj

ifeq ($(V),1)
override V =
endif
ifeq ($(V),0)
override V = @
endif

-include conf/env.mk

LABSETUP ?= ./

TOP = .

GCCPREFIX = 'arm-none-eabi-'


GDBPORT	:= $(shell expr `id -u` % 5000 + 25000)

CC	:= $(GCCPREFIX)gcc -pipe
AS	:= $(GCCPREFIX)as
AR	:= $(GCCPREFIX)ar
LD	:= $(GCCPREFIX)ld
OBJCOPY	:= $(GCCPREFIX)objcopy
OBJDUMP	:= $(GCCPREFIX)objdump
NM	:= $(GCCPREFIX)nm

NCC	:= gcc $(CC_VER) -pipe
NATIVE_CFLAGS := $(CFLAGS) $(DEFS) $(LABDEFS) -I$(TOP) -MD -Wall
TAR	:= gtar
PERL	:= perl

CFLAGS := $(CFLAGS) $(DEFS) $(LABDEFS) -O1 -fno-builtin -I$(TOP) -MD
CFLAGS += -fno-omit-frame-pointer
CFLAGS += -O2 -march=armv7-a -mtune=cortex-a7
CFLAGS += -Wall -Wno-format -Wno-unused -Werror -gstabs -nostdlib -fpic -nostdinc -ffreestanding -lgcc 
CFLAGS += -fno-tree-ch

CFLAGS += $(shell $(CC) -fno-stack-protector -E -x c /dev/null >/dev/null 2>&1 && echo -fno-stack-protector)

LDFLAGS := 

ULDFLAGS := -T user/user.ld

GCC_LIB := $(shell $(CC) $(CFLAGS) -print-libgcc-file-name)

OBJDIRS :=

all:

.SUFFIXES:

.DELETE_ON_ERROR:

.PRECIOUS: %.o $(OBJDIR)/kern/%.o \
	   $(OBJDIR)/lib/%.o $(OBJDIR)/fs/%.o $(OBJDIR)/net/%.o \
	   $(OBJDIR)/user/%.o

KERN_CFLAGS := $(CFLAGS) -DJOS_KERNEL -gstabs 
USER_CFLAGS := $(CFLAGS) -DJOS_USER -gstabs

include kern/Makefrag

QEMU := ~/Desktop/arm-lab/qemu-pi/qemu/build/arm-softmmu/qemu-system-arm
QEMUOPTS = -kernel ./obj/kern/kernel -M raspi2 -serial mon:stdio -gdb tcp::$(GDBPORT)
QEMUOPTS += $(shell if $(QEMU) -nographic -help | grep -q '^-D '; then echo '-D qemu.log'; fi)
//IMAGES = $(OBJDIR)/kern/kernel.img
QEMUOPTS += $(QEMUEXTRA)

.gdbinit: .gdbinit.tmpl
	sed "s/localhost:1234/localhost:$(GDBPORT)/" < $^ > $@

gdb:
	arm-none-eabi-gdb -x .gdbinit

pre-qemu: .gdbinit

qemu: $(IMAGES) pre-qemu
	$(QEMU) $(QEMUOPTS)

qemu-gdb: $(IMAGES) pre-qemu
	@echo "***"
	@echo "*** Now run 'make gdb'." 1>&2
	@echo "***"
	$(QEMU) $(QEMUOPTS) -S

print-qemu:
	@echo $(QEMU)

print-gdbport:
	@echo $(GDBPORT)


# For deleting the build
clean:
	rm -rf $(OBJDIR) jos.in qemu.log



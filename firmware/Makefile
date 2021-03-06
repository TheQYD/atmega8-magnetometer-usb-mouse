### Microcontroller and programmer configuration ###

# $(GCC_MCU) is passed to GCC
GCC_MCU = atmega8

# $(AVRDUDE_MCU) is passed to avrdude
AVRDUDE_MCU = atmega8

# Microcontroller clock
F_CPU = 12000000

# For parallel port programmer:
#AVRDUDE_PARAMS = -c bsd -P /dev/parport0 -E noreset
# For USBasp
AVRDUDE_PARAMS = -c usbasp

# Should we reserve space for the bootloader?
# This option also sets the correct FUSE bytes.
BOOTLOADER_ENABLED = 0

# BOOTLOADER_ADDRESS is 1800 for 8k devices, 3800 for 16k and 7800 for 32k.
# The datasheet says the start address is at 0xC00, but that is indexed in
# words. Multiplying that number by 2 we get 0x1800, indexed in bytes.
BOOTLOADER_ADDRESS = 1800

# In order to fit into ATmega8 8K program space, some pieces of the firmware
# must be disabled.
ENABLE_MOUSE = 1
ENABLE_KEYBOARD = 1
ENABLE_FULL_MENU = 0

# ENABLE_MOUSE:
#   Enables the mouse-emulation code. Required if you want the firmware to work
#   as a mouse.
# ENABLE_KEYBOARD:
#   Enables the keyboard-emulation code and the built-in configuration menu.
#   Required for configuring the device. Also useful for testing and
#   development.
# ENABLE_FULL_MENU:
#   If disabled, removes a few less important items from the built-in menus.
#   Only makes sense when ENABLE_KEYBOARD is 1.
#
#
# Little table of firmware size, as of revision next to 309:a13540b0c33f
#
# MOUSE  KEYBOARD  FULL_MENU  "make combine"  "make all"
#   0       0         0/1       2792 bytes    2870 bytes  (useless)
#   0       1          0        5544 bytes    5492 bytes
#   0       1          1        5852 bytes    6016 bytes
#   1       0         0/1       5540 bytes    5644 bytes
#   1       1          0        8172 bytes    8180 bytes  (no space for bootloader)
#   1       1          1       !8462 bytes   !8700 bytes  (doesn't fit into 8K)
#
#
# Too many choices? I'll make this simple for you, just answer these questions:
#
# * How much memory does your device have?
# |-> More than 8K, it's better than ATmega8
# |   * Then enable everything and don't worry about size!
# '-> Exactly 8K, it's the ATmega8
#     * Go to the next question.
#
# * Do you want a bootloader?
# |-> YES, I want the bootloader!
# |   * Disable the mouse support and write the firmware with keyboard support
# |     and the full menu.
# |   * Use the built-in menu to configure the device.
# |   * After you're done with the configuration, enable the mouse support and
# |     disable the keyboard support, and rewrite the firmware.
# |   * If you need to reconfigure, you need to repeat all these steps again.
# '-> NO, I don't need a bootloader!
#     * Enable the mouse and the keyboard support, but disable the full menu.
#     * Enjoy! It fits into 8K.


### Configurations that depend on the value of BOOTLOADER_ENABLED ###

# FUSE bytes
#
# Note: lfuse should work as either 0x9F or 0xEF
# Note: hfuse should be 0xC0 if bootloader is enabled
#                    or 0xC1 if bootloader is disabled
#
# Tip: This site is handy for calculating the fuse bytes:
#      http://www.engbedded.com/fusecalc/
#
# If you prefer, avrdude also supports binary (0b prefix), hexadecimal
# (0x prefix), octal (0 prefix) and decimal (no prefix)

ifeq ($(BOOTLOADER_ENABLED), 1)
AVRDUDE_PARAMS_FUSE = -U hfuse:w:0xC0:m -U lfuse:w:0x9F:m
else
AVRDUDE_PARAMS_FUSE = -U hfuse:w:0xC1:m -U lfuse:w:0x9F:m
endif

# Maximum firmware size
# ATmega8 has 8K of flash ROM, but 1024 words (2048 bytes) are reserved for the
# bootloader
ifdef BUILDING_BOOTLOADER
 CHECKSIZE_CODELIMIT = 2048
else
 ifeq ($(BOOTLOADER_ENABLED), 1)
  CHECKSIZE_CODELIMIT = 6144
 else
  CHECKSIZE_CODELIMIT = 8192
 endif
endif

# List of objects
ifdef BUILDING_BOOTLOADER
VUSBOBJS = $(VUSBDIR)/usbdrvasm.o $(VUSBDIR)/oddebug.o
MYOBJS = 
else
VUSBOBJS = $(VUSBDIR)/usbdrvasm.o $(VUSBDIR)/oddebug.o $(VUSBDIR)/usbdrv.o
MYOBJS = buttons.o int_eeprom.o keyemu.o mouseemu.o menu.o sensor.o avr315/TWI_Master.o
endif

ALLOBJS = $(PROGNAME).o $(VUSBOBJS) $(MYOBJS)


### Compiling tools configuration ###

AS  = avr-as
CC  = avr-gcc
CXX = avr-g++

NM      = avr-nm
OBJCOPY = avr-objcopy
OBJDUMP = avr-objdump
SIZE    = avr-size

AVRDUDE = avrdude

AVRDUDE_PARAMS += -p $(AVRDUDE_MCU)

ifeq ($(BOOTLOADER_ENABLED), 1)
# -D  = Disable auto erase for flash memory
#       This is required in order to write main firmware to flash without
#       erasing the bootloader.
AVRDUDE_PARAMS += -D
endif

# PROGNAME is the main project file (without extension)
# VUSBDIR is the V-USB driver source-code directory
# CHECKSIZE is the path to the checksize script
ifdef BUILDING_BOOTLOADER
PROGNAME  = bootloader
VUSBDIR   = ../vusb-20100715/usbdrv
CHECKSIZE = ../checksize
else
PROGNAME  = main
VUSBDIR   = ./vusb-20100715/usbdrv
CHECKSIZE = ./checksize
endif

# Starting with simple, straight-forward CFLAGS:
CFLAGS   = -mmcu=$(GCC_MCU) -DF_CPU=$(F_CPU)
CFLAGS  += -DBOOTLOADER_ENABLED=$(BOOTLOADER_ENABLED)
CFLAGS  += -DENABLE_MOUSE=$(ENABLE_MOUSE)
CFLAGS  += -DENABLE_KEYBOARD=$(ENABLE_KEYBOARD)
CFLAGS  += -DENABLE_FULL_MENU=$(ENABLE_FULL_MENU)
CFLAGS  += -std=c99 -pipe -Os -Wall
CFLAGS  += -I./ -I$(VUSBDIR)

# And other FLAGs as well
CXXFLAGS = $(CFLAGS)
ASFLAGS  = -Wa,-adhlns=$(subst $(suffix $<),.lst,$<)
LDFLAGS  = -Wl,-Map=$(PROGNAME).map
LIBS     = -lm

ifdef BUILDING_BOOTLOADER
LDFLAGS += -Wl,--section-start=.text=$(BOOTLOADER_ADDRESS)
endif


### Compiler fine-tuning ###

# -Wno-pointer-sign because those "char*" and "uchar*" warnings are useless.
CFLAGS  += -Wno-pointer-sign

# With a non-returning void main, we can safely ignore this warning.
CFLAGS  += -Wno-main

# People say these are good flags:
CFLAGS  += -funsigned-char -funsigned-bitfields -fpack-struct -fshort-enums

# Warn if the compiler adds padding inside any struct. This will break some
# code assumptions when loading/storing partial sensor data from EEPROM.
# -Wpadded
#   Warn if padding is included in a structure, either to align an element of
#   the structure or to align the whole structure.  Sometimes when this
#   happens it is possible to rearrange the fields of the structure to reduce
#   the padding and so make the structure smaller.
CFLAGS  += -Wpadded

# -Wmissing-field-initializers
#     Warn if a structure's initializer has some fields missing.
CFLAGS  += -Wmissing-field-initializers

# This saved 112 bytes
CFLAGS  += -fno-split-wide-types

# And I'm not very sure about enabling these:
#CFLAGS  += -fno-move-loop-invariants -fno-tree-scev-cprop -fno-inline-small-functions
# But these saved 118 bytes:
CFLAGS  += -fno-move-loop-invariants -fno-tree-scev-cprop

# From GCC manpage:
#   -mcall-prologues
#      Functions prologues/epilogues expanded as call to appropriate subroutines.
#      Code size will be smaller.
# However, in my experiment it actually increased the code size by 54~86 bytes.
#CFLAGS  += -mcall-prologues

# From GCC manpage:
#   -mtiny-stack
#     Change only the low 8 bits of the stack pointer.
# This saves only 10 bytes.
CFLAGS  += -mtiny-stack

# -fms-extensions is required to accept anonymous structures and unions.
CFLAGS  += -fms-extensions

# Supposedly, -ffreestanding together with a non-returning main would save a
# few bytes. But in fact this flag increases the firmware by 8 or 14 bytes,
# even with a non-returning main.
# Also, it gives a linking error about 'fabs'.
#CFLAGS  += -ffreestanding

# Setting the cost of inlining small functions.
# My gcc 4.5.2 doesn't support inline-call-cost, maybe it was removed in newer
# GCC versions?
# Also, enabling those other options increased the firmware size.
#CFLAGS  += --param inline-call-cost=2 -finline-limit=3 -fno-inline-small-functions
#CFLAGS  += -finline-limit=3 -fno-inline-small-functions

# Replace CALL statements with RCALL where possible to save a few bytes.
LDFLAGS += -Wl,--relax

# Don't include unused functions and data.
CFLAGS  += -ffunction-sections -fdata-sections
LDFLAGS += -Wl,--gc-sections -Wl,--print-gc-sections

# Compile all *.c files at once, allowing for better optimizations.
# Note: -combine has been removed in GCC 4.6, in favor of LTO
#       http://gcc.gnu.org/bugzilla/show_bug.cgi?id=29171#c7
#       http://gcc.gnu.org/wiki/LinkTimeOptimization
COMBINE_FLAGS = -combine -fwhole-program

# See this page for more compiler optimisation suggestions:
# http://www.tty1.net/blog/2008-04-29-avr-gcc-optimisations_en.html


#LDFLAGS += $(GENTOO_LD_PATH_WORKAROUND)
#GENTOO_LD_PATH_WORKAROUND = -L/usr/i686-pc-linux-gnu/avr/lib
#GENTOO_LD_PATH_WORKAROUND = -L/usr/x86_64-pc-linux-gnu/avr/lib
#
# See:
# http://bugs.gentoo.org/show_bug.cgi?id=147155
# http://www.avrfreaks.net/index.php?name=PNphpBB2&file=viewtopic&p=336170
#
# ln -snf ../../lib/binutils/avr/2.*/ldscripts/ /usr/avr/lib/ldscripts


### Make targets ###

#Basic rules
.PHONY: all normal-build combine combine-build post-build help clean boot writeboot writeflash writeeeprom writefuse erase dump comments size

all: normal-build post-build

combine: combine-build post-build

normal-build: $(ALLOBJS)
	$(CC) $(CPPFLAGS) $(CFLAGS) $(LDFLAGS) -o $(PROGNAME).elf $^ $(LIBS)

combine-build: $(VUSBOBJS)
# Combined compiling does not work for VUSB source-code. Thus, we add VUSBOBJS
# as dependencies of this target.
	$(CC) $(CPPFLAGS) $(CFLAGS) $(COMBINE_FLAGS) $(LDFLAGS) \
		-o $(PROGNAME).elf \
		$(PROGNAME).c $(MYOBJS:.o=.c) \
		$(VUSBOBJS) $(LIBS)

post-build: $(PROGNAME).elf $(PROGNAME).hex $(PROGNAME).eep $(PROGNAME).lss
	$(CHECKSIZE) $(PROGNAME).elf $(CHECKSIZE_CODELIMIT)

help:
	@echo 'make all         - Builds the project'
	@echo 'make combine     - Compiles all *.c at the same time, allowing some compiler optimizations'
	@echo 'make clean       - Deletes all built files'
	@echo
	@echo 'make boot        - Builds the bootloader (please run "make clean" before)'
	@echo 'make writeboot   - Writes the bootloader to flash memory (please run "make clean" after)'
	@echo
	@echo 'make writeflash  - Writes the flash memory of the microcontroller'
	@echo 'make writeeeprom - Writes the EEPROM of the microcontroller'
	@echo 'make writefuse   - Writes the fuse bytes of the microcontroller'
	@echo 'make erase       - Performs a chip erase'
	@echo 'make dump        - Dumps all memory from the microcontroller'
	@echo
	@echo 'make comments    - Prints all TODO/FIXME/XXX comments'
	@echo 'make size        - Prints the size of all functions/symbols'

clean:
	rm -f $(PROGNAME).{o,s,elf,hex,eep,lss,sym,lst,map}
ifndef BUILDING_BOOTLOADER
	rm -f $(ALLOBJS)
	rm -f $(ALLOBJS:.o=.s)
	rm -f $(ALLOBJS:.o=.lst)
	rm -f $(ALLOBJS:.o=.map)
	cd bootloader && $(MAKE) -f ../Makefile BUILDING_BOOTLOADER=1 clean
endif

boot:
	cd bootloader && $(MAKE) -f ../Makefile BUILDING_BOOTLOADER=1 all

writeboot:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		-U flash:w:bootloader/bootloader.hex:i

writeflash:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		-U flash:w:$(PROGNAME).hex:i

writeeeprom:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		-U eeprom:w:$(PROGNAME).eep:i

writefuse:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		$(AVRDUDE_PARAMS_FUSE)

erase:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		-e
dump:
	$(AVRDUDE) $(AVRDUDE_PARAMS) \
		-U flash:r:flash.dump:i \
		-U eeprom:r:eeprom.dump:i \
		-U hfuse:r:hfuse.dump:b \
		-U lfuse:r:lfuse.dump:b \
		-U lock:r:lock.dump:b \
		-U signature:r:signature.dump:h
# efuse is available on newer AVR microcontrollers
#		-U efuse:r:efuse.dump:b

comments:
	ack --nomake 'TODO|FIXME|XXX'
# I could have used grep, but ack is so much easier and prettier!
# http://betterthangrep.com/
# http://search.cpan.org/dist/ack/
#
# Note: Debian (and Ubuntu?) users have this tool installed as "ack-grep"

size:
# -t d  =>  use decimal radix
# -A    =>  also print the filename
# Sample output:
# main.o:00001546 T main
#
# It's possible to either pass *.o or *.elf
	$(NM) -f bsd -t d -A --size-sort $(PROGNAME).elf | \
		sed 's/^\([^:]\+\):\([0-9a-fA-F]\+\) \(.\) \(.\+\)$$/\2 \3 \4 [\1]/' | \
		sort -n


# Dependencies
# Note: Header dependencies for individual objects are not listed here.
#       Please run "make clean" after you edit any header.
$(PROGNAME).s: $(PROGNAME).c usbconfig.h $(MYOBJS:.o=.h)
$(PROGNAME).o: $(PROGNAME).c usbconfig.h $(MYOBJS:.o=.h)

# This has been commented-out in order to support combined compiling.
#$(PROGNAME).elf: $(ALLOBJS)


# The variables:
# $@ - The name of the target of the rule.
# $? - The names of all the prerequisites that are newer than the target.
# $< - The name of the first prerequisite.
# $^ - The names of all the prerequisites.

# Pattern-rules:
%.s: %.c
	$(CC) -S $(CPPFLAGS) $(CFLAGS) $(ASFLAGS) -o $@ $<
%.o: %.c
	$(CC) -c $(CPPFLAGS) $(CFLAGS) $(ASFLAGS) -o $@ $<
%.o: %.cpp
	$(CXX) -c $(CPPFLAGS) $(CXXFLAGS) $(ASFLAGS) -o $@ $<
%.o: %.S
	$(CC) -c $(CPPFLAGS) $(CFLAGS) $(ASFLAGS) -o $@ $<
# "-x assembler-with-cpp" should not be necessary since this is the default
# file type for the .S (with capital S) extension. However, upper case
# characters are not always preserved on Windows. Add this flag to ensure
# WinAVR compatibility.

%.o: %.h

#%.elf: %.o
#	$(CC) $(CPPFLAGS) $(CFLAGS) $(LDFLAGS) -o $@ $^

# Create final output files (.hex, .eep) from ELF output file.
# Use -O ihex for Intel Hex format
# Use -O binary for binary format
%.hex: %.elf
	$(OBJCOPY) -O ihex -R .eeprom $< $@
#	$(OBJCOPY) -j .text -j .data -O ihex $< $@
%.eep: %.elf
	$(OBJCOPY) -j .eeprom --set-section-flags=.eeprom="alloc,load" \
	--change-section-lma .eeprom=0 -O ihex $< $@

# Create extended listing file from ELF output file.
%.lss: %.elf
	$(OBJDUMP) -h -S -C $< > $@

# Create a symbol table from ELF output file.
%.sym: %.elf
	$(NM) -n $< > $@


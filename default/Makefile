###############################################################################
# Makefile for the project Zooming Secretary
###############################################################################

## General Flags
PROJECT = ZoomingSecretary
GAME = ZMSECRTY
MCU = atmega644
TARGET = $(GAME).elf
CC = avr-gcc
INFO=../gameinfo.properties
UZEBIN_DIR=../../../bin



## Escape spaces in mixer path (due to custom sounds.inc)
EMPTY:=
SPACE:= $(EMPTY) $(EMPTY)
SPACE_ESC:= \\$(SPACE)
MIX_PATH:= $(realpath ../data/sounds.inc)
MIX_PATH_ESC:= $(subst $(SPACE),$(SPACE_ESC),$(MIX_PATH))



## Kernel settings
KERNEL_DIR = ../../../kernel
KERNEL_OPTIONS  = -DINTRO_LOGO=0 -DTRUE_RANDOM_GEN=1
KERNEL_OPTIONS += -DSOUND_MIXER=1 -DSOUND_CHANNEL_5_ENABLE=1 -DMUSIC_ENGINE=STREAM
KERNEL_OPTIONS += -DVIDEO_MODE=3 -DSCREEN_TILES_V=25 -DFIRST_RENDER_LINE=36 -DMAX_SPRITES=24 -DRAM_TILES_COUNT=37 -DTRANSLUCENT_COLOR=0xCE
KERNEL_OPTIONS += -DRESOLUTION_EXT=1 -DVRAM_TILES_H=32


## Options common to compile, link and assembly rules
COMMON = -mmcu=$(MCU)



## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -Wall -gdwarf-2 -std=gnu99 -DF_CPU=28636360UL -Os -fsigned-char -ffunction-sections -fno-toplevel-reorder -mcall-prologues -ffreestanding -fomit-frame-pointer -fno-tree-switch-conversion
CFLAGS += -MD -MP -MT $(*F).o -MF dep/$(@F).d 
CFLAGS += $(KERNEL_OPTIONS)


## Assembly specific flags
ASMFLAGS = $(COMMON)
ASMFLAGS += $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags
LDFLAGS = $(COMMON)
LDFLAGS += -Wl,-Map=$(GAME).map 
LDFLAGS += -Wl,-gc-sections
LDFLAGS += -T $(KERNEL_DIR)/uzebox_linkscript.x



## Intel Hex file production flags
HEX_FLASH_FLAGS = -R .eeprom
HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings



## Objects that must be built in order to link
OBJECTS = uzeboxVideoEngineCore.o uzeboxCore.o uzeboxSoundEngine.o uzeboxSoundEngineCore.o uzeboxVideoEngine.o $(GAME).o 



## Objects explicitly added by the user
LINKONLYOBJECTS = 



## Include Directories
INCLUDES = -I"$(KERNEL_DIR)" 

RESOURCES = ../data/music/mus_level.inc ../data/music/mus_game.inc ../data/music/mus_nobonus.inc ../data/music/mus_welldone.inc ../data/music/mus_dream.inc ../data/music/mus_gameover.inc ../data/music/mus_clear.inc ../data/music.inc

## Build
all: $(RESOURCES) $(TARGET) $(GAME).hex $(GAME).eep $(GAME).lss $(GAME).uze  size



## Rebuild graphics resources
../data/tiles.inc: ../data/tiles.png ../data/tiles.xml
	$(UZEBIN_DIR)/gconvert ../data/tiles.xml

../data/sprites.inc: ../data/sprites.png ../data/sprites.xml
	$(UZEBIN_DIR)/gconvert ../data/sprites.xml

## Rebuild music resources
../data/music/mus_level.inc: ../data/midi/levelsong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/levelsong.mid ../data/music/mus_level.inc -vmus_level -f1

../data/music/mus_game.inc: ../data/midi/gamesong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/gamesong.mid ../data/music/mus_game.inc -vmus_game -f24 -s0 -e30720

../data/music/mus_nobonus.inc: ../data/midi/nobonussong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/nobonussong.mid ../data/music/mus_nobonus.inc -vmus_nobonus -f60

../data/music/mus_welldone.inc: ../data/midi/welldonesong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/welldonesong.mid ../data/music/mus_welldone.inc -vmus_welldone -f18

../data/music/mus_dream.inc: ../data/midi/dreamsong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/dreamsong.mid ../data/music/mus_dream.inc -vmus_dream -f8 -s0 -e7680

../data/music/mus_gameover.inc: ../data/midi/gameoversong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/gameoversong.mid ../data/music/mus_gameover.inc -vmus_gameover -f20

../data/music/mus_clear.inc: ../data/midi/clearsong.mid
	$(UZEBIN_DIR)/midiconv ../data/midi/clearsong.mid ../data/music/mus_clear.inc -vmus_clear -f30

../data/music.inc: ../data/compress-music.cfg
	$(UZEBIN_DIR)/mconvert ../data/compress-music.cfg


## Compile Kernel files
uzeboxVideoEngineCore.o: $(KERNEL_DIR)/uzeboxVideoEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxSoundEngineCore.o: $(KERNEL_DIR)/uzeboxSoundEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxCore.o: $(KERNEL_DIR)/uzeboxCore.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxSoundEngine.o: $(KERNEL_DIR)/uzeboxSoundEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxVideoEngine.o: $(KERNEL_DIR)/uzeboxVideoEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<



## Compile game sources
$(GAME).o: ../$(GAME).c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<



##Link
$(TARGET): $(OBJECTS)
	 $(CC) $(LDFLAGS) $(OBJECTS) $(LINKONLYOBJECTS) $(LIBDIRS) $(LIBS) -o $(TARGET)

%.hex: $(TARGET)
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS)  $< $@

%.eep: $(TARGET)
	-avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

%.lss: $(TARGET)
	avr-objdump -h -S $< > $@

%.uze: $(TARGET)
	-$(UZEBIN_DIR)/packrom $(GAME).hex $@ $(INFO)

UNAME := $(shell sh -c 'uname -s 2>/dev/null || echo not')
AVRSIZEFLAGS := -A ${TARGET}
ifneq (,$(findstring MINGW,$(UNAME)))
AVRSIZEFLAGS := -C --mcu=${MCU} ${TARGET}
endif

size: ${TARGET}
	@echo
	@avr-size -C --mcu=${MCU} ${TARGET}



## Clean target
.PHONY: clean
clean:
	-rm -rf $(OBJECTS) $(GAME).* dep/* *.uze



## Other dependencies
-include $(shell mkdir dep 2>/dev/null) $(wildcard dep/*)


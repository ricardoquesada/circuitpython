# This file is part of the MicroPython project, http://micropython.org/
#
# The MIT License (MIT)
#
# SPDX-FileCopyrightText: Copyright (c) 2020 Scott Shawcroft for Adafruit Industries
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# Select the board to build for.
ifeq ($(BOARD),)
  $(error You must provide a BOARD parameter)
else
  ifeq ($(wildcard boards/$(BOARD)/.),)
    $(error Invalid BOARD specified)
  endif
endif

# If the flash PORT is not given, use the default /dev/tty.SLAB_USBtoUART.
PORT ?= /dev/tty.SLAB_USBtoUART

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk
# Board-specific
include boards/$(BOARD)/mpconfigboard.mk
# Port-specific
include mpconfigport.mk

# CircuitPython-specific
include $(TOP)/py/circuitpy_mpconfig.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

include $(TOP)/supervisor/supervisor.mk

# Include make rules and variables common across CircuitPython builds.
include $(TOP)/py/circuitpy_defns.mk

CROSS_COMPILE = xtensa-esp32s2-elf-

#######################################
# CFLAGS
#######################################

INC += -I.
INC += -I../..
INC += -I$(BUILD)
INC += -I$(BUILD)/genhdr
INC += -I./boards
INC += -I./boards/$(BOARD)
INC += -I./peripherals
INC += -I../../lib/mp-readline
INC += -I../../lib/tinyusb/src
INC += -I../../supervisor/shared/usb
INC += -Iesp-idf/components/freertos/include/freertos
INC += -Iesp-idf/components/freertos/xtensa/include
INC += -Iesp-idf/components/esp32s2/include
INC += -Iesp-idf/components/xtensa/esp32s2/include
INC += -Iesp-idf/components/esp_common/include
INC += -Iesp-idf/components/esp_ringbuf/include
INC += -Iesp-idf/components/esp_rom/include
INC += -Iesp-idf/components/xtensa/include
INC += -Iesp-idf/components/esp_timer/include
INC += -Iesp-idf/components/soc/include
INC += -Iesp-idf/components/soc/src/esp32s2/include
INC += -Iesp-idf/components/soc/soc/include
INC += -Iesp-idf/components/soc/soc/esp32s2/include
INC += -Iesp-idf/components/heap/include
INC += -Iesp-idf/components/esp_system/include
INC += -I$(BUILD)/esp-idf/config

CFLAGS += -DHAVE_CONFIG_H \
          -DMBEDTLS_CONFIG_FILE=\"mbedtls/esp_config.h\" \
          -DUNITY_INCLUDE_CONFIG_H -DWITH_POSIX

# Make our canary value match FreeRTOS's
# This define is in FreeRTOS as tskSTACK_FILL_BYTE 0xa5U which we expand out to a full word.
CFLAGS += -DSTACK_CANARY_VALUE=0xa5a5a5a5

#Debugging/Optimization
ifeq ($(DEBUG), 1)
  CFLAGS += -ggdb
  OPTIMIZATION_FLAGS ?= -Og
  # You may want to enable these flags to make setting breakpoints easier.
  # CFLAGS += -fno-inline -fno-ipa-sra
else
  CFLAGS += -DNDEBUG -ggdb3
  OPTIMIZATION_FLAGS ?= -O2
  # TODO: Test with -flto
  ### CFLAGS += -flto
endif

# option to override compiler optimization level, set in boards/$(BOARD)/mpconfigboard.mk
CFLAGS += $(OPTIMIZATION_FLAGS)

CFLAGS += $(INC) -Werror -Wall -mlongcalls -std=gnu11 -Wl,--gc-sections $(BASE_CFLAGS) $(C_DEFS) $(CFLAGS_MOD) $(COPT)

LDFLAGS = $(CFLAGS) -Wl,-nostdlib  -Wl,-Map=$@.map -Wl,-cref
LDFLAGS += -L$(BUILD)/esp-idf/esp-idf/esp32s2 \
           -Tesp32s2_out.ld \
           -L$(BUILD)/esp-idf/esp-idf/esp32s2/ld \
           -Tesp32s2.project.ld \
           -Lesp-idf/components/esp32s2/ld \
           -Tesp32s2.peripherals.ld \
           -Lesp-idf/components/esp_rom/esp32s2/ld \
           -Tesp32s2.rom.ld \
           -Tesp32s2.rom.libgcc.ld \
           -Tesp32s2.rom.newlib-data.ld \
           -Tesp32s2.rom.newlib-funcs.ld \
           -Tesp32s2.rom.spiflash.ld
LIBS := -lgcc -lc

#

LDFLAGS +=  -Wl,-Bstatic \
			-Wl,--no-warn-mismatch \
			-Wl,--build-id=none \
			-fno-rtti

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

# TinyUSB defines
CFLAGS += -DCFG_TUSB_MCU=OPT_MCU_ESP32S2 -DCFG_TUSB_OS=OPT_OS_FREERTOS -DCFG_TUD_CDC_RX_BUFSIZE=1024 -DCFG_TUD_CDC_TX_BUFSIZE=1024 -DCFG_TUD_MSC_BUFSIZE=4096 -DCFG_TUD_MIDI_RX_BUFSIZE=128 -DCFG_TUD_MIDI_TX_BUFSIZE=128


######################################
# source
######################################


SRC_C += \
	background.c \
	fatfs_port.c \
	mphalport.c \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	modules/$(CIRCUITPY_MODULE).c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/ccsbcs.c \
	lib/timeutils/timeutils.c \
	lib/utils/buffer_helper.c \
	lib/utils/context_manager_helpers.c \
	lib/utils/interrupt_char.c \
	lib/utils/pyexec.c \
	lib/utils/stdout_helpers.c \
	lib/utils/sys_stdio_mphal.c \
	peripherals/pins.c \
	peripherals/rmt.c \
	supervisor/shared/memory.c

ifneq ($(USB),FALSE)
SRC_C += lib/tinyusb/src/portable/espressif/esp32s2/dcd_esp32s2.c
endif

SRC_S =

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
                          $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
                          $(addprefix common-hal/, $(SRC_COMMON_HAL))

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE_INTERNAL))


ifneq ($(FROZEN_MPY_DIR),)
FROZEN_MPY_PY_FILES := $(shell find -L $(FROZEN_MPY_DIR) -type f -name '*.py')
FROZEN_MPY_MPY_FILES := $(addprefix $(BUILD)/,$(FROZEN_MPY_PY_FILES:.py=.mpy))
endif

OBJ += $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.S=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

$(BUILD)/$(FATFS_DIR)/ff.o: COPT += -Os
$(filter $(PY_BUILD)/../extmod/vfs_fat_%.o, $(PY_O)): COPT += -Os

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_MOD) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED)
# Sources that only hold QSTRs after pre-processing.
SRC_QSTR_PREPROCESSOR +=

# IDF build commands
IDF_PATH = $(realpath ./esp-idf)
# create the directory
$(BUILD)/esp-idf:
	$(Q)$(MKDIR) -p $@

# create the config headers
$(BUILD)/esp-idf/config/sdkconfig.h: boards/$(BOARD)/sdkconfig | $(BUILD)/esp-idf
	IDF_PATH=$(IDF_PATH) cmake -S . -B $(BUILD)/esp-idf -DSDKCONFIG=$(BUILD)/esp-idf/sdkconfig -DSDKCONFIG_DEFAULTS="sdkconfig.defaults;boards/$(BOARD)/sdkconfig" -DCMAKE_TOOLCHAIN_FILE=$(IDF_PATH)/tools/cmake/toolchain-esp32s2.cmake -DIDF_TARGET=esp32s2 -GNinja

# build a lib
# Adding -d explain -j 1 -v to the ninja line will output debug info
#$(BUILD)/esp-idf/esp-idf/%.a: $(BUILD)/esp-idf/config/sdkconfig.h
#	ninja -C $(BUILD)/esp-idf esp-idf/$*.a

$(BUILD)/esp-idf/esp-idf/esp32s2/esp32s2_out.ld: $(BUILD)/esp-idf/config/sdkconfig.h
	ninja -C $(BUILD)/esp-idf esp-idf/esp32s2/esp32s2_out.ld

$(BUILD)/esp-idf/esp-idf/esp32s2/ld/esp32s2.project.ld: $(BUILD)/esp-idf/config/sdkconfig.h
	ninja -C $(BUILD)/esp-idf esp-idf/esp32s2/ld/esp32s2.project.ld


$(BUILD)/esp-idf/partition_table/partition-table.bin: $(BUILD)/esp-idf/config/sdkconfig.h
	IDF_PATH=$(IDF_PATH) ninja -C $(BUILD)/esp-idf partition_table/partition-table.bin

# run menuconfig
menuconfig: $(BUILD)/esp-idf/config
	ninja -C $(BUILD)/esp-idf menuconfig
	diff --old-line-format= --unchanged-line-format= sdkconfig.defaults $(BUILD)/esp-idf/sdkconfig > boards/$(BOARD)/sdkconfig || true

# qstr builds include headers so we need to make sure they are up to date
$(HEADER_BUILD)/qstr.i.last: | $(BUILD)/esp-idf/config/sdkconfig.h

# Order here matters
ESP_IDF_COMPONENTS_LINK = freertos log esp_system esp32s2 bootloader_support pthread esp_timer vfs spi_flash app_update esp_common esp32s2 heap newlib driver xtensa soc esp_ringbuf #

ESP_IDF_COMPONENTS_INCLUDE = driver freertos log soc

INC += $(foreach component, $(ESP_IDF_COMPONENTS_INCLUDE), -Iesp-idf/components/$(component)/include)

ESP_IDF_COMPONENTS_EXPANDED = $(foreach component, $(ESP_IDF_COMPONENTS_LINK), $(BUILD)/esp-idf/esp-idf/$(component)/lib$(component).a)
BINARY_BLOBS = esp-idf/components/xtensa/esp32s2/libhal.a
BINARY_WIFI_BLOBS = libcoexist.a libcore.a libespnow.a libmesh.a libnet80211.a libpp.a librtc.a libsmartconfig.a libphy.a
BINARY_BLOBS += $(addprefix esp-idf/components/esp_wifi/lib/esp32s2/, $(BINARY_WIFI_BLOBS))

ESP_IDF_COMPONENTS_EXPANDED += $(BUILD)/esp-idf/esp-idf/soc/soc/esp32s2/libsoc_esp32s2.a esp-idf/components/xtensa/esp32s2/libhal.a
ESP_AUTOGEN_LD = $(BUILD)/esp-idf/esp-idf/esp32s2/esp32s2_out.ld $(BUILD)/esp-idf/esp-idf/esp32s2/ld/esp32s2.project.ld

FLASH_FLAGS = --flash_mode $(CIRCUITPY_ESP_FLASH_MODE) --flash_freq $(CIRCUITPY_ESP_FLASH_FREQ) --flash_size $(CIRCUITPY_ESP_FLASH_SIZE)

all: $(BUILD)/firmware.bin $(BUILD)/firmware.uf2

.PHONY: esp-idf-stamp
esp-idf-stamp: $(BUILD)/esp-idf/config/sdkconfig.h
	ninja -C $(BUILD)/esp-idf \
		bootloader/bootloader.bin \
		esp-idf/bootloader_support/libbootloader_support.a \
		esp-idf/esp32s2/ld/esp32s2.project.ld \
		esp-idf/esp_system/libesp_system.a \
		esp-idf/freertos/libfreertos.a \
		esp-idf/log/liblog.a \
		esp-idf/xtensa/libxtensa.a

$(BUILD)/firmware.elf: $(OBJ) | esp-idf-stamp
	$(STEPECHO) "LINK $@"
	$(Q)$(CC) -o $@ $(LDFLAGS) $^ $(ESP_IDF_COMPONENTS_EXPANDED) $(BINARY_BLOBS) build-$(BOARD)/esp-idf/esp-idf/newlib/libnewlib.a -u newlib_include_pthread_impl -Wl,--start-group $(LIBS) -Wl,--end-group
	# $(Q)$(SIZE) $@ | $(PYTHON3) $(TOP)/tools/build_memory_info.py $(BUILD)/esp-idf/esp-idf/esp32s2/esp32s2_out.ld

$(BUILD)/circuitpython-firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)esptool.py --chip esp32s2 elf2image $(FLASH_FLAGS) --elf-sha256-offset 0xb0 -o $@ $^
#	$(Q)$(OBJCOPY) -O binary $^ $@
#   $(Q)$(OBJCOPY) -O binary -j .vectors -j .text -j .data $^ $@

$(BUILD)/firmware.bin: $(BUILD)/circuitpython-firmware.bin | esp-idf-stamp
	$(Q)$(PYTHON) ../../tools/join_bins.py $@ 0x1000 $(BUILD)/esp-idf/bootloader/bootloader.bin 0x8000 $(BUILD)/esp-idf/partition_table/partition-table.bin 0x10000 $(BUILD)/circuitpython-firmware.bin

$(BUILD)/firmware.uf2: $(BUILD)/circuitpython-firmware.bin
	$(STEPECHO) "Create $@"
	$(Q)$(PYTHON3) $(TOP)/tools/uf2/utils/uf2conv.py -f 0xbfdd4eee -b 0x0000 -c -o $@ $^

flash: $(BUILD)/firmware.bin
	esptool.py --chip esp32s2 -p $(PORT) --no-stub -b 460800 --before=default_reset --after=hard_reset write_flash $(FLASH_FLAGS) 0x0000 $^

flash-circuitpython-only: $(BUILD)/circuitpython-firmware.bin
	esptool.py --chip esp32s2 -p $(PORT) --no-stub -b 460800 --before=default_reset --after=hard_reset write_flash $(FLASH_FLAGS) 0x10000 $^

include $(TOP)/py/mkrules.mk

# Print out the value of a make variable.
# https://stackoverflow.com/questions/16467718/how-to-print-out-a-variable-in-makefile
print-%:
	@echo $* = $($*)

.PHONY: erase compile flash clean flashAT getFlashInfo listPorts

SDK_VERSION := 3.0.5
SDK_PATH := /Users/alexivanov/ESP8266/ESP8266_NONOS_SDK-$(SDK_VERSION)

XTENSA_PATH := /Users/alexivanov/ESP8266/xtensa-lx106-elf
CC := $(XTENSA_PATH)/bin/xtensa-lx106-elf-gcc # cross-compiler 
AR := $(XTENSA_PATH)/bin/xtensa-lx106-elf-ar 
SIZE := $(XTENSA_PATH)/bin/xtensa-lx106-elf-size

BAUDRATE := 230400
FLASH_SIZE := "4MB-c1"

# arkiss at c crypto driver espnow gcc hal json lwip_536 lwip main mbedtls net80211 phy pp pwm smartconfig ssl upgrade wpa wpa2 wps 

LIBS := c gcc hal phy pp net80211 lwip wpa2 crypto json upgrade mbedtls wps smartconfig airkiss wpa main


OBJ_FILES := $(addprefix build/, $(addsuffix .o, $(basename $(notdir $(wildcard src/*)))))
LIBS := $(addprefix -l,$(LIBS))

build: build/app.out
	@$(SIZE) build/app.out

	@esptool.py elf2image build/app.out --output out/app 
	@echo "\033[0;32mBuild finished \033[0m"

inc/%.h: 
	@echo Missing $@ file 

build/%.o: src/%.c inc/%.h inc/user_config.h
	$(call build-obj)

define build-obj
	@echo "\033[1;33mCompiling "$(notdir $@) "\033[0m"
	$(CC) -Iinc -I$(SDK_PATH)/include -I$(SDK_PATH)/include/json -I$(SDK_PATH)/driver_lib/include/ \
	-g -O0 -std=gnu99 -Wpointer-arith -Wundef -Werror -Wl,-EL \
	-fno-inline-functions -nostdlib -mlongcalls \
	-mtext-section-literals -mno-serialize-volatile -D__ets__ -DICACHE_FLASH  -c $< -o $@
endef

build/app.a: $(OBJ_FILES)
	@echo "\033[1;33mBuild app.a\033[0m"
	@$(AR) cru build/app.a $(OBJ_FILES)

build/app.out: build/app.a
	@echo "\033[0;32mCreate app.out \033[0m"
	@$(CC) -L$(SDK_PATH)/lib -T$(SDK_PATH)/ld/eagle.app.v6.ld -u call_user_start -nostdlib \
	-Wl,--no-check-sections -Wl,-static \
	-Wl,--start-group $(LIBS) -Wl,--end-group \
	build/app.a $(SDK_PATH)/lib/libmain.a -o build/app.out


clean: 
	@rm build/* out/*

listports:
	@ls /dev/tty.*

getflashinfo:
	@esptool.py --baud 240000 --after no_reset flash_id

erase:
	@echo "\033[0;31mErasing \033[0m"
	@esptool.py erase_flash
	
flash: build 
	@esptool.py -b $(BAUDRATE) --after no_reset write_flash --flash_size 4MB-c1 --flash_mode dio 0x00000 out/app0x00000.bin 0x10000 out/app0x10000.bin
	@esptool.py -b $(BAUDRATE) --after no_reset verify_flash --flash_size 4MB-c1 --flash_mode dio 0x00000 out/app0x00000.bin 0x10000 out/app0x10000.bin

flashboot:
	@echo "\033[1;33mFlashing boot \033[0m"
	@esptool.py -b $(BAUDRATE) --after no_reset write_flash --flash_size 4MB-c1 --flash_mode dio 0x3FC000 $(SDK_PATH)/bin/esp_init_data_default_v08.bin

flashAT: erase flashboot
	@esptool.py -b $(BAUDRATE) --after no_reset write_flash --flash_size 4MB-c1 --flash_mode dio 0x00000 $(SDK_PATH)/bin/boot_v1.7.bin 
	@esptool.py -b $(BAUDRATE) write_flash --flash_size 4MB-c1 --flash_mode dio 0x01000 $(SDK_PATH)/bin/at/1024+1024/user1.2048.new.5.bin



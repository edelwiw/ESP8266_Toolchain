.PHONY: erase flash clean flashboot getFlashInfo listPorts

BAUDRATE := 230400
FLASH_SIZE := "4MB-c1"

SDK_VERSION := 2.1.1
SDK_PATH := /Users/alexivanov/ESP8266/ESP8266_RTOS_SDK-$(SDK_VERSION)

SDK_INCLUDES := extra_include include driver_lib/include include/espressif include/lwip include/lwip/ipv4 include/lwip/ipv6 include/nopoll include/spiffs include/ssl include/json include/openssl
LIBS := cirom crypto espconn espnow freertos gcc hal lwip main mesh mirom net80211 nopoll phy pp mbedtls openssl smartconfig wpa wps airkiss

XTENSA_PATH := /Users/alexivanov/ESP8266/xtensa-lx106-elf
CC := $(XTENSA_PATH)/bin/xtensa-lx106-elf-gcc # cross-compiler 
AR := $(XTENSA_PATH)/bin/xtensa-lx106-elf-ar 
LD := $(XTENSA_PATH)/bin/xtensa-lx106-elf-gcc 
SIZE := $(XTENSA_PATH)/bin/xtensa-lx106-elf-size

CC_FLAGS := -g -O0 -std=gnu99 -Wpointer-arith -Wundef -Werror -Wl,-EL \
			-fno-inline-functions -nostdlib -mlongcalls \
			-mtext-section-literals -mno-serialize-volatile -D__ets__ \
			-DICACHE_FLASH 


LIBS := $(addprefix -l,$(LIBS))
SDK_INCLUDES := $(addprefix -I$(SDK_PATH)/,$(SDK_INCLUDES))
OBJ_FILES := $(addprefix build/, $(addsuffix .o, $(basename $(notdir $(wildcard src/*)))))

define build-obj
	@echo "\033[1;33mCompiling "$(notdir $@) "\033[0m"
	@$(CC) -Iinc -I$(SDK_PATH)/include -I$(SDK_PATH)/include/json -I$(SDK_PATH)/include/espressif $(SDK_INCLUDES) $(CC_FLAGS) -c $< -o $@
endef

inc/%.h: 
	@echo Missing $@ file 

build/%.o: src/%.c inc/%.h inc/user_config.h
	$(call build-obj)

build/app.a: $(OBJ_FILES)
	@echo "\033[1;33mBuild app.a\033[0m"
	@$(AR) cru build/app.a $(OBJ_FILES)

build/app.out: build/app.a
	@echo "\033[1;33mBuild app.out\033[0m"
	@$(LD) -L$(SDK_PATH)/lib -T$(SDK_PATH)/ld/"eagle.app.v6.ld" -nostdlib -Wl,--no-check-sections -u call_user_start -Wl,-static -Wl,--start-group $(LIBS) build/app.a $(SDK_PATH)/lib/libdriver.a -Wl,--end-group -o build/app.out


build: build/app.out
	@$(SIZE) build/app.out
	@esptool.py elf2image build/app.out --output out/app 
	@echo "\033[0;32mBuild finished \033[0m"

# OUT_FILES := $(wildcard out/*.bin)
# OUT_ADDR := $(patsubst app%,$(notdir %),$(basename $(notdir $(OUT_FILES))))
# FLASH_LIST := $(foreach index, $(shell seq 1 $(words $(OUT_ADDR))), $(word $(index), $(OUT_ADDR)) $(word $(index), $(OUT_FILES)))


flash: build   
	@echo "\033[0;32mFlashing " $(FLASH_LIST) "\033[0m"

	@esptool.py -b $(BAUDRATE) --after no_reset write_flash --flash_size 4MB-c1 --flash_mode dio 0x00000 out/app0x00000.bin 0x20000 out/app0x20000.bin 
	@esptool.py -b $(BAUDRATE) --after no_reset verify_flash --flash_size 4MB-c1 --flash_mode dio 0x00000 out/app0x00000.bin 0x20000 out/app0x20000.bin 

flashboot: 
	@echo "\033[1;33mFlashing boot \033[0m"
	@esptool.py -b $(BAUDRATE) --after no_reset write_flash --flash_size 4MB-c1 --flash_mode dio 0x3FC000 $(SDK_PATH)/bin/esp_init_data_default.bin

clean: 
	@rm build/* out/*

listports:
	@ls /dev/tty.*

getflashinfo:
	@esptool.py -b $(BAUDRATE) --after no_reset flash_id

erase:
	@echo "\033[0;31mErasing \033[0m"
	@esptool.py erase_flash
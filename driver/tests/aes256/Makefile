RISCV_TOOLCHAIN_PATH ?= /opt/riscv-gnu-toolchain
VORTEX_RT_PATH ?= $(wildcard ../../../runtime)

# 0 = ECB encrypt, 1 = ECB decrypt,
# 2 = CBC encrypt, 3 = CBC decrypt,
# 4 = CTR encrypt, 5 = CTR decrypt
# 6 = key expansion for encrypt, 7 = key expansion for decrypt
CRYPTO_OP_TYPE ?= 4
# 16 threads * 8192 blocks/thread * 16B/block = 2MiB
# See aes_op_type_t in common.h for the possible values for t
OPTS ?= -n 8192 -t $(CRYPTO_OP_TYPE)
# NATIVE, HYBRID, NATIVE,HYBRID, TABLE, TABLE,MONOTABLE, or SOFTWARE
# (although SOFTWARE is currently unused)
CRYPTO_ACCEL_MODE ?= NATIVE

VX_CC  = $(RISCV_TOOLCHAIN_PATH)/bin/riscv32-unknown-elf-gcc
VX_CXX = $(RISCV_TOOLCHAIN_PATH)/bin/riscv32-unknown-elf-g++
VX_DP  = $(RISCV_TOOLCHAIN_PATH)/bin/riscv32-unknown-elf-objdump
VX_CP  = $(RISCV_TOOLCHAIN_PATH)/bin/riscv32-unknown-elf-objcopy

comma=,
VX_CFLAGS += -march=rv32imf -mabi=ilp32f -O3 -Wstack-usage=1024 -ffreestanding -nostartfiles -fdata-sections -ffunction-sections -Wall -Werror -Wextra
VX_CFLAGS += -B$(RISCV_TOOLCHAIN_PATH)/riscv32-unknown-elf/bin -I$(VORTEX_RT_PATH)/include -I$(VORTEX_RT_PATH)/../hw -DAES_$(subst $(comma), -DAES_,$(CRYPTO_ACCEL_MODE))

VX_LDFLAGS += -Wl,-Bstatic,-T,$(VORTEX_RT_PATH)/linker/vx_link.ld -Wl,--gc-sections $(VORTEX_RT_PATH)/libvortexrt.a

VX_SRCS = kernel.c aes256-impl.c tables.c

#CXXFLAGS += -std=c++11 -O2 -Wall -Wextra -pedantic -Wfatal-errors
CXXFLAGS += -std=c++11 -O0 -g -Wall -Wextra -pedantic -Wfatal-errors

CXXFLAGS += -I../../include

PROJECT = aes256

SRCS = aes256.cpp
PROJ_LIBS = $(shell pkg-config --libs openssl)

all: $(PROJECT) kernel.bin kernel.dump
 
kernel.dump: kernel.elf
	$(VX_DP) -D kernel.elf > kernel.dump

kernel.bin: kernel.elf
	$(VX_CP) -O binary kernel.elf kernel.bin

kernel.elf: $(VX_SRCS)
	$(VX_CC) $(VX_CFLAGS) $(VX_SRCS) $(VX_LDFLAGS) -o kernel.elf

$(PROJECT): $(SRCS)
	$(CXX) $(CXXFLAGS) $^ $(LDFLAGS) -L../../stub -lvortex $(PROJ_LIBS) -o $@

run-fpga: $(PROJECT)
	LD_LIBRARY_PATH=../../opae:$(LD_LIBRARY_PATH) ./$(PROJECT) $(OPTS)

run-asesim: $(PROJECT)
	ASE_LOG=0 LD_LIBRARY_PATH=../../opae/ase:$(LD_LIBRARY_PATH) ./$(PROJECT) $(OPTS)

run-vlsim: $(PROJECT)
	LD_LIBRARY_PATH=../../opae/vlsim:$(LD_LIBRARY_PATH) ./$(PROJECT) $(OPTS)

run-rtlsim: $(PROJECT)
	LD_LIBRARY_PATH=../../rtlsim:$(LD_LIBRARY_PATH) ./$(PROJECT) $(OPTS)
	
run-simx: $(PROJECT)
	LD_LIBRARY_PATH=../../simx:$(LD_LIBRARY_PATH) ./$(PROJECT) $(OPTS)

.depend: $(SRCS)
	$(CXX) $(CXXFLAGS) -MM $^ > .depend;

clean:
	rm -rf $(PROJECT) *.o .depend

clean-all: clean
	rm -rf *.elf *.bin *.dump

ifneq ($(MAKECMDGOALS),clean)
    -include .depend
endif

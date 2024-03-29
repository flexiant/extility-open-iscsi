# This Makefile will work only with GNU make.

OSNAME=$(shell uname -s)

# allow users to override these
# eg to compile for a kernel that you aren't currently running
KERNELRELEASE ?= $(shell uname -r)
KSRC ?= /lib/modules/$(KERNELRELEASE)/build

KSUBLEVEL=$(shell cat $(KSRC)/Makefile | awk -F= '/^SUBLEVEL =/ {print $$2}' | \
			sed 's/^[ \t]*//;s/[ \t]*$$//')

ifeq ($(OSNAME),Linux)
	ifeq ($(KSUBLEVEL),11)
		IPC_CFLAGS=-DNETLINK_ISCSI=12 -D_GNU_SOURCE
	else
	ifeq ($(KSUBLEVEL),12)
		IPC_CFLAGS=-DNETLINK_ISCSI=12 -D_GNU_SOURCE
	else
		IPC_CFLAGS=-DNETLINK_ISCSI=8 -D_GNU_SOURCE
	endif
	endif
IPC_OBJ=netlink.o
else
ifeq ($(OSNAME),FreeBSD)
IPC_CFLAGS=
IPC_OBJ=ioctl.o
endif
endif

OPTFLAGS ?= -O2 -g
WARNFLAGS ?= -Wall -Wstrict-prototypes
CFLAGS += $(OPTFLAGS) $(WARNFLAGS) -I../include -I. -D$(OSNAME) $(IPC_CFLAGS)
PROGRAMS = iscsid iscsiadm iscsistart

# libc compat files
SYSDEPS_SRCS = $(wildcard ../utils/sysdeps/*.o)
# sources shared between iscsid, iscsiadm and iscsistart
ISCSI_LIB_SRCS = util.o io.o auth.o login.o log.o md5.o sha1.o iface.o idbm.o sysfs.o host.o session_info.o iscsi_sysfs.o $(SYSDEPS_SRCS)
# sources shared between iscsid and iscsiadm
COMMON_SRCS =  $(ISCSI_LIB_SRCS)
# core initiator files
INITIATOR_SRCS = initiator.o scsi.o actor.o event_poll.o mgmt_ipc.o isns.o \
		cxgb3i.o transport.o
# fw boot files
FW_BOOT_SRCS = $(wildcard ../utils/fwparam_ibft/*.o)

all: $(PROGRAMS)

iscsid: $(COMMON_SRCS) $(IPC_OBJ) $(INITIATOR_SRCS) iscsid.o
	$(CC) $(CFLAGS) $^ -o $@

iscsiadm: $(COMMON_SRCS) $(FW_BOOT_SRCS) strings.o discovery.o iscsiadm.o
	$(CC) $(CFLAGS) $^ -o $@

iscsistart: $(IPC_OBJ) $(ISCSI_LIB_SRCS) $(INITIATOR_SRCS) $(FW_BOOT_SRCS) \
		iscsistart.o statics.o
	$(CC) $(CFLAGS) -static $^ -o $@
clean:
	rm -f *.o $(PROGRAMS) .depend $(LIBSYS)

depend:
	gcc $(CFLAGS) -M `ls *.c` > .depend

-include .depend

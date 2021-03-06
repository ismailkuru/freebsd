# $FreeBSD$

HAVE_GELI=yes

.include <bsd.init.mk>

.PATH:		${BOOTSRC}/i386/boot2 ${BOOTSRC}/i386/common ${SASRC}

FILES=		zfsboot
MAN=		zfsboot.8

NM?=		nm

BOOT_COMCONSOLE_PORT?= 0x3f8
BOOT_COMCONSOLE_SPEED?= 9600
B2SIOFMT?=	0x3

REL1=	0x700
ORG1=	0x7c00
ORG2=	0x2000

CFLAGS+=-DBOOTPROG=\"zfsboot\" \
	-O1 \
	-DZFS -DBOOT2 \
	-DSIOPRT=${BOOT_COMCONSOLE_PORT} \
	-DSIOFMT=${B2SIOFMT} \
	-DSIOSPD=${BOOT_COMCONSOLE_SPEED} \
	-I${LDRSRC} \
	-I${BOOTSRC}/i386/common \
	-I${BOOTSRC}/i386 \
	-I${ZFSSRC} \
	-I${SYSDIR}/cddl/boot/zfs \
	-I${BTXLIB} \
	-I${BOOTSRC}/i386/boot2 \
	-Wall -Waggregate-return -Wbad-function-cast -Wno-cast-align \
	-Wmissing-declarations -Wmissing-prototypes -Wnested-externs \
	-Wpointer-arith -Wshadow -Wstrict-prototypes -Wwrite-strings \
	-Winline

CFLAGS.gcc+=	--param max-inline-insns-single=100
.if ${MACHINE} == "amd64"
LIBZFSBOOT=${BOOTOBJ}/zfs32/libzfsboot.a
.else
LIBZFSBOOT=${BOOTOBJ}/zfs/libzfsboot.a
.endif

LD_FLAGS+=${LD_FLAGS_BIN}

CLEANFILES+=	zfsboot

zfsboot: zfsboot1 zfsboot2
	cat zfsboot1 zfsboot2 > zfsboot

CLEANFILES+=	zfsboot1 zfsldr.out zfsldr.o

zfsboot1: zfsldr.out
	${OBJCOPY} -S -O binary zfsldr.out ${.TARGET}

zfsldr.out: zfsldr.o
	${LD} ${LD_FLAGS} -e start -Ttext ${ORG1} -o ${.TARGET} zfsldr.o

CLEANFILES+=	zfsboot2 zfsboot.ld zfsboot.ldr zfsboot.bin zfsboot.out \
		zfsboot.o zfsboot.s zfsboot.s.tmp sio.o cons.o drv.o

# We currently allow 256k bytes for zfsboot - in practice it could be
# any size up to 3.5Mb but keeping it fixed size simplifies zfsldr.
# 
BOOT2SIZE=	262144

zfsboot2: zfsboot.ld
	@set -- `ls -l ${.ALLSRC}`; x=$$((${BOOT2SIZE}-$$5)); \
	    echo "$$x bytes available"; test $$x -ge 0
	${DD} if=${.ALLSRC} of=${.TARGET} obs=${BOOT2SIZE} conv=osync

zfsboot.ld: zfsboot.ldr zfsboot.bin ${BTXKERN}
	btxld -v -E ${ORG2} -f bin -b ${BTXKERN} -l zfsboot.ldr \
	    -o ${.TARGET} -P 1 zfsboot.bin

zfsboot.ldr:
	cp /dev/null ${.TARGET}

zfsboot.bin: zfsboot.out
	${OBJCOPY} -S -O binary zfsboot.out ${.TARGET}

zfsboot.out: ${BTXCRT} zfsboot.o sio.o drv.o cons.o
	${LD} ${LD_FLAGS} -Ttext ${ORG2} -o ${.TARGET} ${.ALLSRC} ${LIBZFSBOOT} ${LIBGELIBOOT} ${LIBSA32}

SRCS=	zfsboot.c

.include <bsd.prog.mk>

# XXX: clang integrated-as doesn't grok .codeNN directives yet
CFLAGS.zfsldr.S=	${CLANG_NO_IAS}

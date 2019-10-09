OBJ/UPURSFS: upursfs.txt 6502src.txt compile_date
	beebasm -i $< -o $@ -v > $@.log.txt

OBJ/HOSTFS: hostfs.txt 6502src.txt compile_date
	beebasm -i $< -o $@ -v > $@.log.txt

OBJ/HOSTFSM: hostfs_myelin.txt 6502src.txt compile_date
	beebasm -i $< -o $@ -v > $@.log.txt

all: OBJ/HOSTFS OBJ/UPURSFS OBJ/HOSTFSM

compile_date: upursfs.txt 6502src.txt hostfs.txt hostfs_myelin.txt
	(echo MACRO COMPILE_DATE ; echo "   EQUS \":`date +%Y%m%d-%H%M%S`\"" ; echo ENDMACRO ) > compile_date
	grep EQUS compile_date

remake: OBJ/UPURSFS
	rm compile_date
	make $<

.PHONY : check publish clean

clean:
	/bin/rm -f OBJ/* compile_date

check: OBJ/HOSTFS
	hcmp OBJ/HOSTFS jgh_original_HostFS.ROM 0x8000

publish: UPURSFS/source.zip UPURSFS/UPURSFS.ROM UPURSFS/UPURSFS.log.txt UPURSFS/README.txt

UPURSFS/source.zip: 6502src.txt Makefile hostfs.txt upursfs.txt
	/bin/rm -f UPURSFS/source.zip
	zip UPURSFS/source.zip $+

UPURSFS/UPURSFS.ROM: OBJ/UPURSFS
	cp -a $< $@

UPURSFS/UPURSFS.log.txt: OBJ/UPURSFS.log.txt
	cp -a $< $@

UPURSFS/README.txt: README.txt
	cp -a $< $@

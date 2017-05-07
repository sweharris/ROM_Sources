SRC=ROMS/* README MANUAL1 MENU manager.src solidisk.src ctory.src beebem.src generic.src manual.txt Original_Solidisk_ROM 6502src.txt upursfs.txt hostfs.txt compile_date

TARGET=2M128

ALL.ROMS: ROMS/MANAGER ROMS/CTORY ROMS/BEEBEM ROMS/GENERIC

ROMS/MANAGER: solidisk.src compile_date
	beebasm -i solidisk.src -o ROMS/MANAGER -v > ROMS/MANAGER.log

ROMS/CTORY: ctory.src compile_date
	beebasm -i ctory.src -o ROMS/CTORY

ROMS/BEEBEM: beebem.src compile_date hostfs.txt
	beebasm -i beebem.src -o ROMS/BEEBEM

ROMS/GENERIC: generic.src compile_date
	beebasm -i generic.src -o ROMS/GENERIC

_done: $(TARGET)/2m128.tar $(TARGET)/index.html $(TARGET)/manual.txt
	touch _done

6502src.txt: ../HostFS/6502src.txt
	cp $^ $@

# For this ROM we're gonna stick the UPURS on a second user port
upursfs.txt: ../HostFS/upursfs.txt
	sed -e 's/SWEH_EXTENSION = TRUE/SWEH_EXTENSION = 200/' -e 's/USER_PORT = .*/USER_PORT = \&FCB1/' < $^ > $@

hostfs.txt: ../HostFS/hostfs.txt
	sed -e 's/SWEH_EXTENSION = FALSE/SWEH_EXTENSION = 200/' -e 's/DEFAULT_FS = .*/DEFAULT_FS = FALSE/' -e 's/BreakKey = .*/BreakKey = \&35  \\ U/' < $^ > $@

compile_date: upursfs.txt 6502src.txt
	(echo MACRO COMPILE_DATE ; echo "   EQUS \":`date +%Y%m%d-%H%M%S`\"" ; echo ENDMACRO ) > compile_date
	grep EQUS compile_date

$(TARGET)/manual.txt: manual.txt
	cp $^ $@

$(TARGET)/2m128.tar: $(SRC)
	rm -f $@
	tar cf $@ $^

$(TARGET)/index.html: README
	./build_index $< > $@

clean:
	rm -f 6502src.txt upursfs.txt hostfs.txt compile_date *done ROMS/*

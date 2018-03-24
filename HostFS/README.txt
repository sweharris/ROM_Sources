The latest version of this code is always at
  http://sweh.spuddy.org/Beeb/UPURSFS/

There is a working GIT repo (which may be ahead of the published version)
in the "HostFS" directory of
  https://github.com/sweharris/ROM_Sources

This is what happens when you take the TubeHost concept
  http://sweh.spuddy.org/Beeb/TubeHost/
and combine it with UPURS
  http://www.retro-kit.co.uk/UPURS/

Summary: connect the BBC User port to a PC serial port and then run
a filesystem over it!

=====

Basically, I took the HostFS code written by JGH:
  http://mdfs.net/Software/Tube/Serial/
  http://mdfs.net/Software/Tube/Serial/6502.src

I converted this to BeebASM format source code
  http://www.retrosoftware.co.uk/wiki/index.php/BeebAsm

To verify I hadn't broken JGH's code in the process I confirmed I
could assemble a ROM that was bytewise identical to his original.

Then I hacked in routines provide by MartinB, mangled in such a way
that they remained "API" compatible with JGH's code.  

Basically, blame me for the resulting mess, and not either of the
others, who unknowingly sacrificed their code to my vicious knife.
Bwahahahah.

The result... works :-)

For straight file transfers I've been able to send a 24K file from my
PC to the Beeb in around 4.5 seconds.  That's not bad, at all.  Gives
us around 43Kbit/s transfer speeds which, if my math is correct, isn't
too far off floppy disk speeds back in the day...

* In theory a BBC floppy did 300RPM; a track was 10 sectors of 256 bytes,
* so 2.5K in .2 seconds is 2.5*5*8=100Kbit/s.  However it was rare for
* the Beeb to handle that; worst case it would read one sector then miss
* the next so we'd have to wait for a rotation... 10Kbit/s!  Interleaved
* disks might get us 30 to 50Kbit/s.

43Kbit/s is perfectly reasonable.

However there is a "gotcha".  Using the user port to act at 115200 baud
is pushing the Beeb to its limits.  Basically there's around 17 clock
cycles per bit.  This means it's sometimes tempermental.  In particular,
detecting the start bit of a new byte is not always successful in a
_polling_ setup (which Serial Tube kinda is; some parts can't just block
waiting for input).

To help work around this, Martin's code puts in some wait states; after
asserting CTS if there's no start bit then wait a while, and check again.
If there's still no start bit then there's no data waiting for us; give
up (here is where blocking works better; just wait forever until we get
a start bit).  The "gotcha" is with this delay.  The Serial Tube concept
is byte-driven, and some routines (eg BGET#) are very very byte driven.
So while a *LOAD can be fast (we just throw data until we're finished),
something like a *TYPE is inherently slow; it opens the file then for
each character sends a unique command to HostFS.  This means we can
never stream, and the delay kicks in.

Through trial and error, I've found a delay value that works for me.  I
can "*TYPE" a 1K file without data loss or freeze up.  It's not _fast_
but it works.  It's good enough for *EXEC functionality!  However, if
you find hangs on random character based I/O like this then it might
be that the delay loop is too short.  If you look in the code then
you'll see

  .wait   LDY     #20
  .wait1  LDX     #0
  .wait2  DEX
          BNE     wait2
          DEY
          BNE     wait1
          RTS

Martin originally had "LDY #0"; for me #20 seems to work.  You might want
to tune this.  If you can't recompile then the accompanying "log" file
will show you where to binary edit the ROM:

  .wait
       8BEC   A0 14      LDY #&14
  .wait1
       8BEE   A2 00      LDX #&00
  .wait2
       8BF0   CA         DEX
       8BF1   D0 FD      BNE &8BF0
       8BF3   88         DEY
       8BF4   D0 F8      BNE &8BEE
       8BF6   60         RTS

Here, 8BED holds the value to edit.  The actual location may change,
depending on code versions, so always look at the log file if you
want to make binary edits.  (NOTE: I've found that if I put my TubeHost
code into heavy debug mode then data loss may appear, so this tuning is
a balancing act)

(I told you I made nasty changes to the code!  It's all my fault...)

The code does require a buffer to handle the speed of data from the PC;
I'm using the NMI space &D10->&D70.  In theory this shouldn't be used
by anything but programs might...  Other possible options are commented
in the code.

=====

Other thoughts...

  U-BREAK selects UPURSFS

=====

JGH has released his code under an "attribution" clause

   \ J.G.Harston, 16-Nov-2010
   \ This code may be freely reused, with acknowledgements

MartinB's code has been released under a "I trust you" agreement.  Heh.

As a result, I'm not sure the resulting code is GPLv2 compliant.  So
I'm releasing the combined mess under an "attribution clause" as well.

I've made the ROM copyright read (after comments from JGH)

            EQUS "(C)"
    IF      UPURS
            EQUS "Stephen Harris (sweh), with code from MartinB, HostFS core"
    ELSE
            EQUS "J.G.Harston"
    ENDIF

Although this code can assembled to generate an original HostFS ROM, I
strongly recommend you go to JGH's original site if that's what you want;
he'll always have the latest version with bug fixes etc.

The TubeHost side has been tested with an Asus EEEbox (Atom N270
based CPU @1.6Ghz) running Ubuntu 12.10 with an FTDI FT231X adapter,
and with a home built Core i5 2.66Ghz machine running CentOS 6 with
an FTDI FT2232C adapter (note: it loses data if DEBUG is set too high;
appears to be totally reliable with DEBUG set low enough).  The UPURSFS
client has been tested on an issue 7 model B

I don't have a 2nd processor, so I'm not sure what'll happen with that.
JGH has code which looks like it handles the case, but I'm utterly unable
to test.

BREAK/SHIFT-BREAK/CONTROL-BREAK
===============================
State is maintained on the server, so pressing BREAK doesn't change
anything.  If you press SHIFT+BREAK then all file handles are closed,
dir and lib is set to :0.$ and an indicator is sent back to the client
as of the OPT4 status of drive 0.  With a modified HostFS client this
may allow boot.  The changes I've made allow LOAD/RUN/EXEC functionality
at boot time; this is experimental.

Now, in theory, control-break should also send an update to the server,
but if the serial port is not connected or if the TubeHost software is
not running or baud rates are wrong then this causes a hang at power up.
To this end the client doesn't let the server know that control-break has
been sent.  This means TubeHost is subtly different in behaviour to disks.

SWEH EXTENSIONS
===============
In the source code you'll see references to SWEH_EXTENSION, SWEH_BREAK
and similar.  If you define SWEH_EXTENSION FALSE then you'll get the
behaviour as listed above.  SWEH_EXTENSION TRUE enables some change
to this...
* Remove the character you pressed (eg U-BREAK).
* Send Shift-Break to the server
* If SWEH_BREAK set, also send Control-Break (power on) to the server
* Add message " (No RTS)" if RTS isn't detected to *HELP and BREAK

Now the shift/ctrl-BREAK behaviour also depends on RTS.  If RTS is present
(and, on UPURS, if the top bit isn't floating) then we assume the serial
or UPURS cable is plugged in, and something is ready to accept data.
This isn't necessarily correct (eg Linux may have RTS set even if there's
no process) so the Beeb might hang on a hard boot.  In my mind this is
kinda similar to ADFS hanging if there's no floppy, so the UPURSFS ROM
image provided has these values set.  

Good luck!


=========================
0.04  2013/06/03
  At boot time, if no RTS is present then print a message but do not select
  us; that allows another filesystem to take over.  In my case I have
  TurboMMC and UPURSFS both plugged in; if the UPURS cable is in and the
  HostFS active then UPURSFS is selected.  If MMC is plugged in then
  the DFS is selected.  Magic!   (SWEH_EXTENSION option)

(No version change; no code impact...)
  SWEH_EXTENSION == 200  ---> being called from RAM_Manager ROM code to
    be included in that ROM.  Don't do this unless you know what you're
    doing!

0.05 2013/06/29
  SWEH_BREAK should also handle shift-break

0.06 2013/10/20
  Not really a version change but...  allow the UPURS USER_PORT base to
  be changed so it can work with 2nd user port
    http://sweh.spuddy.org/Beeb/2nd_User_Port/

0.07 2013/11/05
  DEFAULT_FS - if set then this ROM will attempt to select itself as a
    filesystem on BREAK (assuming no higher ROM already done this).
    If clear then BreakKey+BREAK (eg TAB-BREAK or U-BREAK) can still be
    done to select this ROM as a filesystem

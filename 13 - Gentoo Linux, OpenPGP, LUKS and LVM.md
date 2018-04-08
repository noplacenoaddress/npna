![*the gentoo penguins*](https://upload.wikimedia.org/wikipedia/commons/a/a3/Gentoo_Penguin_AdF.jpg)

In our journey, in our adventure, in our war with the *privacy cannibals* we use to find or be found by a few good fellows. This time we use another base system operative, we start to be helped by [**Gentoo Linux**](https://www.gentoo.org/).

This is a very special, historic distribution, his goal is that there's no precompile binary and all the system is optimized for the host hardware. The result is a ultra fast operative system, like the pet that represent it: the [gentoo *rapid swimming* penguin](https://en.wikipedia.org/wiki/Gentoo_penguin). 

Speaking about those penguins, i want to do a little parenthesis. I want to speak about [**climate changes**](https://en.wikipedia.org/wiki/Climate_change). And i will not use correct words. **I'm furious**. Furious because we're the real plague in earth. We're hable only to speak about money, luxury and power. There's only a little problem, our world, our earth, the nature that live here, the same nature that after a very long evolution have give us, [*homo sapiens sapiens*](https://en.wikipedia.org/wiki/Homo_sapiens), the opportunity **to be **, is rapidly diyng. Because we have not decided to be, we decided to destroy. 

Those little, innocent, funny penguins are dying. Because the medium temperature in their natural enviroment have changed, a lot. It's important to understand that a change in a scale of decimal have got devastating effects. I'm not an expert, but there's many documents that can proove this fact. Look a this paper:

[**IAATO ACCE Fact Sheet**](https://iaato.org/documents/10157/100441/ClimateChangeA4.pdf)

If you don't want to read, simply look at this photos that were taken in the same place with a difference of 100 years, the site is in Artic and not in Antartica but the concept is the same:

![climate change](https://pbs.twimg.com/media/DW_qtWhWAAAFXlX.jpg)

## Speaking about unix, monitorizing and repair SCSI disk

But this is an article about computer science and not about nature, because i'm an IT addicted, nature for me is a passion but i don't have the right knowledge to speak about it.

Let's start with deep configuration, the escenario is that we've got a new harddisk in our **Gentoo** host and we want to dedicate it for guest machines in a **QEMU/KVM** enviroment. But the disk it's not *new*, so we've to check it's hardware integrity; we know that is produced by [Hitachi](https://en.wikipedia.org/wiki/Hitachi):

```sh
taglio@cyberdream ~ $ lsblk --output NAME,MODEL,VENDOR |grep Hitachi
sdb                                                    Hitachi HTS72323 ATA     
taglio@cyberdream ~ $ 
taglio@cyberdream ~ $ sudo blkid | grep sdb
/dev/sdb1: LABEL="Reservado para el sistema" UUID="128A32078A31E7BD" TYPE="ntfs" PARTUUID="410fac6e-01"
/dev/sdb2: UUID="86A83F08A83EF5F1" TYPE="ntfs" PARTUUID="410fac6e-02"
taglio@cyberdream ~ $ 
```

We've found it using `lsblk` and identificate the `UUID` of two active partiions in it using `blkid` with root power. Now let's check errors with `smartctl`:

```sh
taglio@cyberdream ~ $ emerge -s smartmontools
  
[ Results for search key : smartmontools ]
Searching...

*  sys-apps/smartmontools
      Latest version available: 6.6
      Latest version installed: 6.6
      Size of files: 883 KiB
      Homepage:      https://www.smartmontools.org
      Description:   Tools to monitor storage systems to provide advanced warning of disk degradation
      License:       GPL-2

[ Applications found : 1 ]

taglio@cyberdream ~ $
taglio@cyberdream ~ $ sudo emerge -av smartmontools
...
taglio@cyberdream ~ $
taglio@cyberdream ~ $ sudo rc-config add smartd default
Adding smartd to following runlevels
  default                   [done]
taglio@cyberdream ~ $ 
taglio@cyberdream ~ $ sudo rc-config start smartd
Starting init script
smartd            | * Starting smartd ...                                                                                 [ ok ]
taglio@cyberdream ~ $ 
taglio@cyberdream ~ $ sudo smartctl -x /dev/sdb
smartctl 6.6 2017-11-05 r4594 [x86_64-linux-4.9.76-gentoo-r18828] (local build)
Copyright (C) 2002-17, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     Hitachi HTS723232A7A364
Serial Number:    E3834563HMKERN
LU WWN Device Id: 5 000cca 61dd6fc08
Firmware Version: EC2OA60W
User Capacity:    320,072,933,376 bytes [320 GB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ATA8-ACS T13/1699-D revision 6
SATA Version is:  SATA 2.6, 3.0 Gb/s
Local Time is:    Wed Mar  7 16:40:33 2018 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
AAM feature is:   Unavailable
APM level is:     128 (minimum power consumption without standby)
Rd look-ahead is: Enabled
Write cache is:   Disabled
DSN feature is:   Unavailable
ATA Security is:  Disabled, frozen [SEC2]
Wt Cache Reorder: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: FAILED!
Drive failure expected in less than 24 hours. SAVE ALL DATA.
See vendor-specific Attribute list for failed Attributes.

General SMART Values:
Offline data collection status:  (0x00)	Offline data collection activity
					was never started.
					Auto Offline Data Collection: Disabled.
Self-test execution status:      (  73)	The previous self-test completed having
					a test element that failed and the test
					element that failed is not known.
Total time to complete Offline 
data collection: 		(   45) seconds.
Offline data collection
capabilities: 			 (0x51) SMART execute Offline immediate.
					No Auto Offline data collection support.
					Suspend Offline collection upon new
					command.
					No Offline surface scan supported.
					Self-test supported.
					No Conveyance Self-test supported.
					Selective Self-test supported.
SMART capabilities:            (0x0003)	Saves SMART data before entering
					power-saving mode.
					Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
					General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   2) minutes.
Extended self-test routine
recommended polling time: 	 (  78) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
					SCT Error Recovery Control supported.
					SCT Feature Control supported.
					SCT Data Table supported.

SMART Attributes Data Structure revision number: 16
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAGS    VALUE WORST THRESH FAIL RAW_VALUE
  1 Raw_Read_Error_Rate     POSR-K   087   025   062    Past 3342383
  2 Throughput_Performance  P-S--K   100   100   040    -    0
  3 Spin_Up_Time            PO---K   243   100   033    -    1
  4 Start_Stop_Count        -O--CK   096   096   000    -    6522
  5 Reallocated_Sector_Ct   PO--CK   001   001   005    NOW  2307 (0 2079)
  7 Seek_Error_Rate         POSR-K   100   099   067    -    0
  8 Seek_Time_Performance   P-S--K   100   100   040    -    0
  9 Power_On_Hours          -O--CK   069   069   000    -    13986
 10 Spin_Retry_Count        PO--CK   100   100   060    -    0
 12 Power_Cycle_Count       -O--CK   097   097   000    -    5749
183 Runtime_Bad_Block       -O--CK   100   100   000    -    0
184 End-to-End_Error        PO--CK   100   100   097    -    0
187 Reported_Uncorrect      -O--CK   100   006   000    -    251268471652846
188 Command_Timeout         -O--CK   100   001   000    -    3633959802393
190 Airflow_Temperature_Cel -O---K   073   049   045    -    27 (Min/Max 26/27)
191 G-Sense_Error_Rate      -O--CK   001   001   000    -    65755
192 Power-Off_Retract_Count -O--CK   100   100   000    -    10158235
193 Load_Cycle_Count        -O--CK   049   049   000    -    513620
196 Reallocated_Event_Count -O--CK   009   009   000    -    2262
197 Current_Pending_Sector  -O--CK   091   057   000    -    444
198 Offline_Uncorrectable   ----CK   100   100   000    -    0
199 UDMA_CRC_Error_Count    -OS-CK   100   100   000    -    1
223 Load_Retry_Count        -O-R-K   100   100   000    -    0
                            ||||||_ K auto-keep
                            |||||__ C event count
                            ||||___ R error rate
                            |||____ S speed/performance
                            ||_____ O updated online
                            |______ P prefailure warning

General Purpose Log Directory Version 1
SMART           Log Directory Version 1 [multi-sector log support]
Address    Access  R/W   Size  Description
0x00       GPL,SL  R/O      1  Log Directory
0x01           SL  R/O      1  Summary SMART error log
0x02           SL  R/O      1  Comprehensive SMART error log
0x03       GPL     R/O      1  Ext. Comprehensive SMART error log
0x04       GPL     R/O      7  Device Statistics log
0x06           SL  R/O      1  SMART self-test log
0x07       GPL     R/O      1  Extended self-test log
0x09           SL  R/W      1  Selective self-test log
0x10       GPL     R/O      1  NCQ Command Error log
0x11       GPL     R/O      1  SATA Phy Event Counters log
0x80-0x9f  GPL,SL  R/W     16  Host vendor specific log
0xe0       GPL,SL  R/W      1  SCT Command/Status
0xe1       GPL,SL  R/W      1  SCT Data Transfer

SMART Extended Comprehensive Error Log Version: 1 (1 sectors)
Device Error Count: 31665 (device log contains only the most recent 4 errors)
	CR     = Command Register
	FEATR  = Features Register
	COUNT  = Count (was: Sector Count) Register
	LBA_48 = Upper bytes of LBA High/Mid/Low Registers ]  ATA-8
	LH     = LBA High (was: Cylinder High) Register    ]   LBA
	LM     = LBA Mid (was: Cylinder Low) Register      ] Register
	LL     = LBA Low (was: Sector Number) Register     ]
	DV     = Device (was: Device/Head) Register
	DC     = Device Control Register
	ER     = Error register
	ST     = Status register
Powered_Up_Time is measured from power on, and printed as
DDd+hh:mm:SS.sss where DD=days, hh=hours, mm=minutes,
SS=sec, and sss=millisec. It "wraps" after 49.710 days.

Error 31665 [0] occurred at disk power-on lifetime: 13986 hours (582 days + 18 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER -- ST COUNT  LBA_48  LH LM LL DV DC
  -- -- -- == -- == == == -- -- -- -- --
  40 -- 51 00 01 00 00 00 03 2b 97 00 00  Error: UNC at LBA = 0x00032b97 = 207767

  Commands leading to the command that caused the error were:
  CR FEATR COUNT  LBA_48  LH LM LL DV DC  Powered_Up_Time  Command/Feature_Name
  -- == -- == -- == == == -- -- -- -- --  ---------------  --------------------
  60 00 08 00 b0 00 00 00 03 2b 90 40 00     00:36:04.809  READ FPDMA QUEUED
  60 00 08 00 a8 00 00 00 03 2b 88 40 00     00:36:03.852  READ FPDMA QUEUED
  60 00 08 00 a0 00 00 00 03 2b 80 40 00     00:36:03.852  READ FPDMA QUEUED
  60 00 08 00 98 00 00 00 03 2b 78 40 00     00:36:03.851  READ FPDMA QUEUED
  60 00 08 00 90 00 00 00 03 2b 70 40 00     00:36:03.851  READ FPDMA QUEUED

Error 31664 [3] occurred at disk power-on lifetime: 13986 hours (582 days + 18 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER -- ST COUNT  LBA_48  LH LM LL DV DC
  -- -- -- == -- == == == -- -- -- -- --
  40 -- 51 00 4a 00 00 00 03 2b b6 00 00  Error: UNC at LBA = 0x00032bb6 = 207798

  Commands leading to the command that caused the error were:
  CR FEATR COUNT  LBA_48  LH LM LL DV DC  Powered_Up_Time  Command/Feature_Name
  -- == -- == -- == == == -- -- -- -- --  ---------------  --------------------
  60 01 00 00 88 00 00 00 03 2b 00 40 00     00:36:01.150  READ FPDMA QUEUED
  60 01 00 00 80 00 00 00 03 2a 00 40 00     00:36:01.043  READ FPDMA QUEUED
  60 00 f0 00 78 00 00 00 03 29 10 40 00     00:36:01.042  READ FPDMA QUEUED
  60 00 90 00 70 00 00 00 03 28 80 40 00     00:36:01.041  READ FPDMA QUEUED
  60 00 38 00 68 00 00 00 03 28 40 40 00     00:36:01.041  READ FPDMA QUEUED

Error 31663 [2] occurred at disk power-on lifetime: 13986 hours (582 days + 18 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER -- ST COUNT  LBA_48  LH LM LL DV DC
  -- -- -- == -- == == == -- -- -- -- --
  40 -- 51 00 1c 00 00 00 03 2b e4 00 00  Error: UNC at LBA = 0x00032be4 = 207844

  Commands leading to the command that caused the error were:
  CR FEATR COUNT  LBA_48  LH LM LL DV DC  Powered_Up_Time  Command/Feature_Name
  -- == -- == -- == == == -- -- -- -- --  ---------------  --------------------
  60 01 00 00 48 00 00 00 03 2b 00 40 00     00:35:31.764  READ FPDMA QUEUED
  60 01 00 00 40 00 00 00 03 2a 00 40 00     00:35:31.738  READ FPDMA QUEUED
  60 00 f0 00 38 00 00 00 03 29 10 40 00     00:35:31.731  READ FPDMA QUEUED
  60 00 90 00 30 00 00 00 03 28 80 40 00     00:35:31.730  READ FPDMA QUEUED
  60 00 38 00 28 00 00 00 03 28 40 40 00     00:35:31.730  READ FPDMA QUEUED

Error 31662 [1] occurred at disk power-on lifetime: 13985 hours (582 days + 17 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER -- ST COUNT  LBA_48  LH LM LL DV DC
  -- -- -- == -- == == == -- -- -- -- --
  40 -- 51 00 08 00 00 00 03 2b d0 00 00  Error: UNC at LBA = 0x00032bd0 = 207824

  Commands leading to the command that caused the error were:
  CR FEATR COUNT  LBA_48  LH LM LL DV DC  Powered_Up_Time  Command/Feature_Name
  -- == -- == -- == == == -- -- -- -- --  ---------------  --------------------
  60 00 08 00 d0 00 00 00 03 2b d0 40 00     00:01:48.238  READ FPDMA QUEUED
  b0 00 d5 00 01 00 00 00 c2 4f 01 00 00     00:01:47.992  SMART READ LOG
  60 00 08 00 b0 00 00 00 03 2b c8 40 00     00:01:47.905  READ FPDMA QUEUED
  b0 00 d5 00 01 00 00 00 c2 4f 06 00 00     00:01:47.659  SMART READ LOG
  60 00 08 00 90 00 00 00 03 2b c0 40 00     00:01:47.406  READ FPDMA QUEUED

SMART Extended Self-test Log Version: 1 (1 sectors)
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short offline       Completed: unknown failure    90%     13985         0
# 2  Short offline       Completed: read failure       90%     10168         28292250
# 3  Short offline       Completed: read failure       90%     10168         28292250
# 4  Extended offline    Completed: read failure       90%     10165         353774
# 5  Short offline       Completed: read failure       90%     10165         353774
# 6  Extended offline    Completed without error       00%      9306         -
# 7  Short offline       Completed without error       00%      9304         -
# 8  Extended offline    Completed without error       00%      9304         -
# 9  Short offline       Completed without error       00%      9303         -
#10  Short offline       Completed without error       00%      9275         -
#11  Short offline       Completed without error       00%      9213         -
#12  Short offline       Completed without error       00%      9110         -
#13  Short offline       Completed without error       00%      9095         -
#14  Short offline       Completed without error       00%      9072         -
#15  Short offline       Completed without error       00%      9008         -
#16  Short offline       Completed without error       00%      8959         -
#17  Short offline       Aborted by host               80%      6559         -
#18  Short offline       Completed without error       00%         4         -

SMART Selective self-test log data structure revision number 1
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.

SCT Status Version:                  3
SCT Version (vendor specific):       256 (0x0100)
SCT Support Level:                   1
Device State:                        Active (0)
Current Temperature:                    27 Celsius
Power Cycle Min/Max Temperature:     26/27 Celsius
Lifetime    Min/Max Temperature:      3/51 Celsius
Lifetime    Average Temperature:        32 Celsius
Under/Over Temperature Limit Count:   0/0

SCT Temperature History Version:     2
Temperature Sampling Period:         1 minute
Temperature Logging Interval:        1 minute
Min/Max recommended Temperature:      0/60 Celsius
Min/Max Temperature Limit:           -40/65 Celsius
Temperature History Size (Index):    128 (104)

Index    Estimated Time   Temperature Celsius
 105    2018-03-07 14:33    27  ********
 ...    ..( 13 skipped).    ..  ********
 119    2018-03-07 14:47    27  ********
 120    2018-03-07 14:48    26  *******
 ...    ..( 10 skipped).    ..  *******
   3    2018-03-07 14:59    26  *******
   4    2018-03-07 15:00    27  ********
   5    2018-03-07 15:01    26  *******
   6    2018-03-07 15:02    26  *******
   7    2018-03-07 15:03    27  ********
   8    2018-03-07 15:04    26  *******
   9    2018-03-07 15:05    26  *******
  10    2018-03-07 15:06    27  ********
  11    2018-03-07 15:07    27  ********
  12    2018-03-07 15:08    26  *******
  13    2018-03-07 15:09    27  ********
  14    2018-03-07 15:10    27  ********
  15    2018-03-07 15:11    26  *******
  16    2018-03-07 15:12    26  *******
  17    2018-03-07 15:13    27  ********
  18    2018-03-07 15:14    26  *******
  19    2018-03-07 15:15    27  ********
 ...    ..(  9 skipped).    ..  ********
  29    2018-03-07 15:25    27  ********
  30    2018-03-07 15:26    26  *******
  31    2018-03-07 15:27    26  *******
  32    2018-03-07 15:28    27  ********
  33    2018-03-07 15:29    27  ********
  34    2018-03-07 15:30    27  ********
  35    2018-03-07 15:31    26  *******
  36    2018-03-07 15:32    27  ********
  37    2018-03-07 15:33    27  ********
  38    2018-03-07 15:34    27  ********
  39    2018-03-07 15:35    26  *******
  40    2018-03-07 15:36    27  ********
  41    2018-03-07 15:37    27  ********
  42    2018-03-07 15:38    26  *******
  43    2018-03-07 15:39    27  ********
  44    2018-03-07 15:40    27  ********
  45    2018-03-07 15:41    27  ********
  46    2018-03-07 15:42    26  *******
  47    2018-03-07 15:43    26  *******
  48    2018-03-07 15:44    27  ********
  49    2018-03-07 15:45    27  ********
  50    2018-03-07 15:46    26  *******
  51    2018-03-07 15:47    27  ********
  52    2018-03-07 15:48    26  *******
  53    2018-03-07 15:49    26  *******
  54    2018-03-07 15:50     ?  -
  55    2018-03-07 15:51    27  ********
 ...    ..(  3 skipped).    ..  ********
  59    2018-03-07 15:55    27  ********
  60    2018-03-07 15:56    26  *******
  61    2018-03-07 15:57    27  ********
 ...    ..( 12 skipped).    ..  ********
  74    2018-03-07 16:10    27  ********
  75    2018-03-07 16:11    26  *******
  76    2018-03-07 16:12    27  ********
  77    2018-03-07 16:13    27  ********
  78    2018-03-07 16:14    27  ********
  79    2018-03-07 16:15    26  *******
  80    2018-03-07 16:16    27  ********
 ...    ..( 23 skipped).    ..  ********
 104    2018-03-07 16:40    27  ********

SCT Error Recovery Control:
           Read:     85 (8.5 seconds)
          Write:     85 (8.5 seconds)

Device Statistics (GP Log 0x04)
Page  Offset Size        Value Flags Description
0x01  =====  =               =  ===  == General Statistics (rev 1) ==
0x01  0x008  4            5749  ---  Lifetime Power-On Resets
0x01  0x010  4           13986  ---  Power-on Hours
0x01  0x018  6     23880099276  ---  Logical Sectors Written
0x01  0x020  6       525642514  ---  Number of Write Commands
0x01  0x028  6     44822287873  ---  Logical Sectors Read
0x01  0x030  6       954672703  ---  Number of Read Commands
0x03  =====  =               =  ===  == Rotating Media Statistics (rev 1) ==
0x03  0x008  4           13288  ---  Spindle Motor Power-on Hours
0x03  0x010  4           13250  ---  Head Flying Hours
0x03  0x018  4          513621  ---  Head Load Events
0x03  0x020  4            2079  ---  Number of Reallocated Logical Sectors
0x03  0x028  4         1381497  ---  Read Recovery Attempts
0x03  0x030  4               2  ---  Number of Mechanical Start Failures
0x04  =====  =               =  ===  == General Errors Statistics (rev 1) ==
0x04  0x008  4             494  ---  Number of Reported Uncorrectable Errors
0x04  0x010  4            5657  ---  Resets Between Cmd Acceptance and Completion
0x05  =====  =               =  ===  == Temperature Statistics (rev 1) ==
0x05  0x008  1              27  ---  Current Temperature
0x05  0x010  1              26  N--  Average Short Term Temperature
0x05  0x018  1              33  N--  Average Long Term Temperature
0x05  0x020  1              51  ---  Highest Temperature
0x05  0x028  1               3  ---  Lowest Temperature
0x05  0x030  1              41  N--  Highest Average Short Term Temperature
0x05  0x038  1              24  N--  Lowest Average Short Term Temperature
0x05  0x040  1              38  N--  Highest Average Long Term Temperature
0x05  0x048  1              25  N--  Lowest Average Long Term Temperature
0x05  0x050  4               0  ---  Time in Over-Temperature
0x05  0x058  1              60  ---  Specified Maximum Operating Temperature
0x05  0x060  4               0  ---  Time in Under-Temperature
0x05  0x068  1               0  ---  Specified Minimum Operating Temperature
0x06  =====  =               =  ===  == Transport Statistics (rev 1) ==
0x06  0x008  4           13637  ---  Number of Hardware Resets
0x06  0x010  4            2565  ---  Number of ASR Events
0x06  0x018  4               1  ---  Number of Interface CRC Errors
                                |||_ C monitored condition met
                                ||__ D supports DSN
                                |___ N normalized value

Pending Defects log (GP Log 0x0c) not supported

SATA Phy Event Counters (GP Log 0x11)
ID      Size     Value  Description
0x0001  2            0  Command failed due to ICRC error
0x0002  2            0  R_ERR response for data FIS
0x0003  2            0  R_ERR response for device-to-host data FIS
0x0004  2            0  R_ERR response for host-to-device data FIS
0x0005  2            0  R_ERR response for non-data FIS
0x0006  2            0  R_ERR response for device-to-host non-data FIS
0x0007  2            0  R_ERR response for host-to-device non-data FIS
0x0009  2            2  Transition from drive PhyRdy to drive PhyNRdy
0x000a  2            2  Device-to-host register FISes sent due to a COMRESET
0x000b  2            0  CRC errors within host-to-device FIS
0x000d  2            0  Non-CRC errors within host-to-device FIS

taglio@cyberdream ~ $ 
```

With `-x` we're printing all **SMART** and non-SMART information about the device. It's the same than using `-H -i -g all -A -l error -l selftest -l background -l sasphy` that mean:

1.  `-H` prints the health status of the device.
2.  `-i` *unknown.*
3.  `-g all` get all **non-SMART** device settings.
4.  `-A` For SCSI devices the "attributes" are obtained from the temperature and start-stop  cycle  counter  log pages.   Certain  vendor  specific  attributes are listed if recognised.  The attributes are output in a rela‐tively free format (compared with ATA disk attributes).
5.  `-l error` in `SCSI` prints  the error counter log pages for reads, write and verifies.  The verify row is only output if it has  an element other than zero.
6.  `-l selftest` in `SCSI` It identifies the test that failed and consists of either the  number  of  the segment that failed during the test, or the number of the test that failed and the number of the segment  in which  the  test  was  run,  using  a  vendor-specific method of putting both numbers into a  single  byte.   The  Logical  Block Address (LBA) of the first error is printed in hexadecimal nota‐tion.
7.  `-l background` in `SCSI` he background scan results log outputs information derived from Background Media Scans (BMS) done after power  up  and/or  periodically  (e.g. every 24 hours) on recent SCSI disks.
8.  `-l sasphy` in `SCSI` prints  values  and  descriptions  of  the  SAS  (SSP)  Protocol Specific log page (log page 0x18). 

As we can apreciate from the selftest's results there are various error in this harddisk. Now we put the harddisk in `offline` mode and launch the `long` test:

```sh
taglio@cyberdream ~ $ sudo smartctl -t offline /dev/sdb
taglio@cyberdream ~ $ sudo smartctl -t long /dev/sdb
taglio@cyberdream ~ $ sudo smartctl -A /dev/sdb
```

Now we try to repair bad blocks using an other utility:

```sh
taglio@cyberdream ~ $ sudo emerge -av sys-block/hdrecover
taglio@cyberdream ~ $ sudo hdrecover /dev/sdb
```

recheck the `SMART` status with `smartctl -A` and verify that `Current_Pending_Sector` is now 0 and
`Reallocated_Event_Count` will have risen by the number of sectors the drive decided to reallocate. Remember that `hdrecover` will destroy data.

## New partition table, partition, cipher and LVM

![gnome Parted](https://upload.wikimedia.org/wikipedia/commons/6/64/GParted.png)

Now that we've hopefully repair our disk we can inizialize it:

```sh
taglio@cyberdream ~ $ sudo parted -a optimal /dev/sdb
Password:  
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mktable gpt                                                      
Warning: The existing disk label on /dev/sdb will be destroyed and all data on
this disk will be lost. Do you want to continue?
Yes/No? Yes                                                               
(parted) unit s
(parted) print free                                                       
Model: ATA Hitachi HTS72323 (scsi)
Disk /dev/sdb: 625142448s
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End         Size        File system  Name  Flags
        34s    625142414s  625142381s  Free Space

(parted) mkpart primary 2048s 625141760s
(parted) print                                                            
Model: ATA Hitachi HTS72323 (scsi)
Disk /dev/sdb: 625142448s
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start  End         Size        File system  Name     Flags
 1      2048s  625141760s  625139713s               primary

(parted) quit
```

This is the best way we can create a new `gpt` disk table and a new primary partition. We've to select the correct start and end sector using this formula:

```sh
taglio@cyberdream ~ $ echo "$((((34 + 2047) / 2048) * 2048))s $((625142414 - (625142414 % 2048)))s"
2048s 625141760s
taglio@cyberdream ~ $ 
```

Next we're going to create a `urandom` seed of `8192KiB` that will be encrypted with **OpenPGP**. 

```sh
taglio@cyberdream ~/.gnupg/disk_seed $ dd if=/dev/urandom bs=8388607 count=1 | gpg --symmetric --cipher-algo AES256 --output luks-key.gpg
1+0 records in
1+0 records out
8388607 bytes (8.4 MB, 8.0 MiB) copied, 10.7103 s, 783 kB/s
taglio@cyberdream ~/.gnupg/disk_seed $ du -h luks-key.gpg 
8.1M	luks-key.gpg
taglio@cyberdream ~/.gnupg/disk_seed $ 
```

We've encrypt using the cipher `AES256`, in my machine there's a lot more available:

```sh
taglio@cyberdream ~/.gnupg/disk_seed $ gpg --version
gpg (GnuPG) 2.2.5
libgcrypt 1.8.2
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/taglio/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
taglio@cyberdream ~/.gnupg/disk_seed $ 
```

Next we will pipe our `urandom` seed to `cryptsetup`, the Linux utility to manipulate [`LUKS` ](https://gitlab.com/cryptsetup/cryptsetup/blob/master/README.md). 

```sh
taglio@cyberdream ~/.gnupg/disk_seed $ su
Password: 
cyberdream /home/taglio/.gnupg/disk_seed # gpg --decrypt luks-key.gpg | cryptsetup --cipher serpent-xts-plain64 --key-size 512 --hash whirlpool --key-file - luksFormat /dev/sdb1
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
cyberdream /home/taglio/.gnupg/disk_seed # cryptsetup luksDump /dev/sdb1
LUKS header information for /dev/sdb1

Version:       	1
Cipher name:   	serpent
Cipher mode:   	xts-plain64
Hash spec:     	whirlpool
Payload offset:	4096
MK bits:       	512
MK digest:     	a2 2b 25 4e 6b 24 eb 59 38 be b5 2c 1d c8 ab 2f 79 f2 e3 6b 
MK salt:       	be a5 be c9 40 76 92 bc 1b e7 89 24 56 ec 31 ab 
               	de 44 d7 a4 54 b9 7f 10 ff 33 52 7c fe 35 f9 7f 
MK iterations: 	215250
UUID:          	5416f85d-ea43-4b3e-bb06-d125900145ab

Key Slot 0: ENABLED
	Iterations:         	1726812
	Salt:               	45 07 5a 07 6c 56 5c e8 3d eb 2f 3a a5 e2 7f d8 
	                      	17 a6 cc 35 6a 61 a4 23 c5 1f 87 2a c6 3f d2 b5 
	Key material offset:	8
	AF stripes:            	4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

Let's understand the meaning of `cryptsetup` options:

1.  `--cipher serpent-xts-plain64` we've selected encryption cipher [**serpent**](https://en.wikipedia.org/wiki/Serpent_(cipher)), encryption mode [**xts**](https://en.wikipedia.org/wiki/Disk_encryption_theory#XTS) and Initial Vector (IV) generator **plain64** (*The IV offset is a sector count that is added to the sector number before creating the IV. It can be used to create a map that starts after the first encrypted sector. Usually you'll set it to zero except your device is only partially available or you need to configure some mode compatible with other encryption system.*).
2.  `--key-size 512` sets key size in bits. The argument has to be a multiple of 8. The possible key-sizes are limited by the cipher and mode used.
3.  `--hash whirlpool` Specifies the hash used in the LUKS key setup scheme and volume key digest for luksFormat. The specified hash is used as hash-parameter for PBKDF2 and for the AF splitter. We have select [**whirlpool**](https://en.wikipedia.org/wiki/Whirlpool_(cryptography))
4.  `--key-file -` got the key file from the piped result of `gpg --decrypt`
5.  `luksFormat /dev/sdb1` formats `sdb1` as LUKS device 

With the `luksDump sdb1` command we want to be sure that our `luksFormat` was good as you can see from the output. 

Now we open the encrypted device and create a physical volume and a volume group for [**LVM**](https://en.wikipedia.org/wiki/Logical_Volume_Manager_%28Linux%29).

```sh
cyberdream /home/taglio/.gnupg/disk_seed # echo RELOADAGENT | gpg-connect-agent
OK
cyberdream /home/taglio/.gnupg/disk_seed # gpg --decrypt luks-key.gpg | cryptsetup --key-file - luksOpen /dev/sdb1 virtualrepo
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
cyberdream /home/taglio/.gnupg/disk_seed # ls -al /dev/mapper/virtualrepo 
lrwxrwxrwx 1 root root 7 Mar  8 12:04 /dev/mapper/virtualrepo -> ../dm-4
cyberdream /home/taglio/.gnupg/disk_seed # pvcreate /dev/mapper/virtualrepo 
  Physical volume "/dev/mapper/virtualrepo" successfully created.
cyberdream /home/taglio/.gnupg/disk_seed # vgcreate vg3 /dev/mapper/virtualrepo 
  Volume group "vg3" successfully created
cyberdream /home/taglio/.gnupg/disk_seed # 
```

With the `RELOADAGENT` we indicate to `gpg-agent` to restart it. Next we map the crypto device /dev/sdb1 in the virtual device `/dev/mapper/virtualrepo`.

The last thing we're doing for our **QEMU/KVM** enviroment with direct **LVM** disk access is create a physical volume that will be used to store the volume group on with `pvcreate /dev/mapper/virtualrepo` and the create a volume that will be used to store the logical volumes on with `vgcreate vg3 /dev/mapper/virtualrepo`. More informations about **Gentoo** and **LVM** can be found [here](https://wiki.gentoo.org/wiki/LVM).

Ok, for today is sufficient, it's a pleasure write those unperfect articles for you, 



*kisses, RG*
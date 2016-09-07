---
layout: post
title: Debugging tools
---

Dstat -- Stat on cpu/disk/network/memory/contextswitch -- Best is I get the time as well
  

dstat -t -cdngy
----system---- ----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
  date/time   |usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
07-09 09:16:22| 11   3  86   0   0   0|  11k  490k|   0     0 |   0     0 |6116    15k
07-09 09:16:23|  7   3  90   0   0   0|   0    48k|6314B 4958B|   0     0 |6869    17k
07-09 09:16:24|  8   3  89   0   0   0|   0   180k|  17k   12k|   0     0 |5917    16k
07-09 09:16:25|  5   3  92   0   0   0|   0     0 |6297B 6133B|   0     0 |6540    16k
07-09 09:16:26|  7   2  91   0   0   0|   0     0 |9141B 8274B|   0     0 |6122    16k
07-09 09:16:27|  4   2  94   0   0   0|   0     0 |  15k 7396B|   0     0 |5815    16k
07-09 09:16:28|  5   2  93   0   0   0|   0   164k|  12k   14k|   0     0 |6004    16k
07-09 09:16:29| 33   6  61   0   0   0|   0    68k|  17k   12k|   0     0 |  14k   23k
07-09 09:16:30| 29   8  63   0   0   0|   0    52k|  26k   17k|   0     0 |9241    20k

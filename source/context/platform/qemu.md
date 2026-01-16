# QEMU

## Virt


## 设备

LoongArch平台上的设备怎么添加，mmio设备怎么挂载，等等

以设备为例：

```text
-fda/-fdb file  use 'file' as floppy disk 0/1 image
-hda/-hdb file  use 'file' as hard disk 0/1 image
-hdc/-hdd file  use 'file' as hard disk 2/3 image
-cdrom file     use 'file' as CD-ROM image
-blockdev [driver=]driver[,node-name=N][,discard=ignore|unmap]
          [,cache.direct=on|off][,cache.no-flush=on|off]
          [,read-only=on|off][,auto-read-only=on|off]
          [,force-share=on|off][,detect-zeroes=on|off|unmap]
          [,driver specific parameters...]
                configure a block backend
-drive [file=file][,if=type][,bus=n][,unit=m][,media=d][,index=i]
       [,cache=writethrough|writeback|none|directsync|unsafe][,format=f]
       [,snapshot=on|off][,rerror=ignore|stop|report]
       [,werror=ignore|stop|report|enospc][,id=name]
       [,aio=threads|native|io_uring]
       [,readonly=on|off][,copy-on-read=on|off]
       [,discard=ignore|unmap][,detect-zeroes=on|off|unmap]
       [[,bps=b]|[[,bps_rd=r][,bps_wr=w]]]
       [[,iops=i]|[[,iops_rd=r][,iops_wr=w]]]
       [[,bps_max=bm]|[[,bps_rd_max=rm][,bps_wr_max=wm]]]
       [[,iops_max=im]|[[,iops_rd_max=irm][,iops_wr_max=iwm]]]
       [[,iops_size=is]]
       [[,group=g]]
                use 'file' as a drive image
-mtdblock file  use 'file' as on-board Flash memory image
-sd file        use 'file' as SecureDigital card image
-snapshot       write to temporary files instead of disk image files
```
+++
date = "2016-06-14T13:43:46+12:00"
draft = false
title = "Too many disks?"
+++

I sometimes get confused with how many disks I have connected and which is which.  There is the `lsblk` command which is cool and prints output in a tree format:

```sh
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                     8:0    0 238.5G  0 disk
├─sda1                  8:1    0   243M  0 part /boot
├─sda2                  8:2    0     1K  0 part
└─sda5                  8:5    0 238.2G  0 part
  ├─ubuntu--vg-root   252:0    0 222.3G  0 lvm  /
  └─ubuntu--vg-swap_1 252:1    0    16G  0 lvm  [SWAP]
sdb                     8:16   0 232.9G  0 disk
└─sdb1                  8:17   0 232.9G  0 part
sdc                     8:32   0 465.8G  0 disk
sr0                    11:0    1  1024M  0 rom  
```

However, I can't see the model names of the disks in there.  You can read this stuff out of the `/sys/block` folder structure.  So I made a small ruby script to do that for me:

```ruby
#!/usr/bin/env ruby

disks = Dir["/sys/block/sd*"]

puts "%-12s %-20s %-12s %-8s" % ["LOCATION", "MODEL", "SIZE", "PARTS"]

disks.each do |path|

  disk  = File.basename(path)
  model = File.read("#{path}/device/model").chomp
  ss    = File.read("#{path}/queue/hw_sector_size").chomp.to_f
  size  = ((File.read("#{path}/size").chomp.to_i * ss) / 1024 / 1024 / 1024).round(2)
  parts = Dir["#{path}/#{disk}*/partition"].size

  puts "%-12s %-20s %-12s %-8d" % ["/dev/#{disk}", model, "#{size}GB", parts]
end
```

Now I can get this output by running `lsdisk` (I put it in my `$HOME/bin` folder):

```sh
LOCATION     MODEL                SIZE         PARTS   
/dev/sda     Samsung SSD 850      238.47GB     3       
/dev/sdb     WDC WD2500BEKT-7     232.89GB     1       
/dev/sdc     HGST HCC545050A7     465.76GB     0       
/dev/sdd     Storage Device       14.92GB      3   
```

Much easier to differentiate between them now.  Don't want to `dd` onto the wrong one!

---
layout: post
title:  "Upgrading libreboot on Lenovo Thinkpad T440p"
date:   2024-10-10  13:49:00 +0200
categories: libreboot
---

Purpose of this page is to provide a simple guide to just upgrade to a newer release. In this guide I'm upgrading my Lenovo Thinkpad T440p to version 20241008 internally on Void Linux.

---
{: data-content="setting up lbmk"}

download lbmk [here](https://codeberg.org/libreboot/lbmk/releases), in this example tag _20241008_

if you downloaded a release (tar or zip):
```
git init
```

in extracted _lbmk/_ directory check for supported distributions to install dependencies: 
```
ls config/dependencies/
```

install dependencies: 
```
./mk dependencies void
```

set git username and email:
```
git config user.name "example"
git config user.email example@example.com
```

check for coreboot tree for your board and then build it
```
./mk -b coreboot list
./mk -d coreboot t440plibremrc_12mb
```

---
{: data-content="download, inject and check rom"}

navigate to [release roms mirror](https://mirror.koddos.net/libreboot/testing/20241008/roms/) to find and download rom for your model

inject the rom you downloaded 
```
./vendor inject ../roms/libreboot-20241008_t440plibremrc_12mb.tar.xz
```

you can now find injected roms in _bin/release/t440plibremrc\_12mb/_

now i will copy my rom of choice into another directory and name it _libreboot.rom_: 
```
cp bin/release/t440plibremrc_12mb/seabios_t440plibremrc_12mb_libgfxinit_corebootfb.rom ../roms/new/libreboot.rom
```

check if the rom was injected successfully: 
```
./elf/ifdtool/default/ifdtool -x ../roms/new/libreboot.rom
```
this will create some files, we need to look at the one with _me_ in its name, in this case it's _flashregion\_2\_intel\_me.bin_. run hexdump on it: 
```
hexdump flashregion_2_intel_me.bin
```
check the output, if it contains only ones (ff), is short or does not look like a bunch of code, the rom was not injected correctly. if it's long and has a bunch of random values, intel me firmware was correctly injected


this is how the rom looks if injected incorrectly:
{% include zoomable_image.html img_link="/assets/images/2024-10-10-updating-libreboot/rom-not-injected.png" %}

this is how it looks when correctly injected:
{% include zoomable_image.html img_link="/assets/images/2024-10-10-updating-libreboot/rom-injected.png" %}

---
{: data-content="disable security"}

to write prepared rom we have to disable _/dev/mem_ write protections. open _/etc/default/grub_ with your preferred text editor and add `iomem=relaxed` to _GRUB\_CMDLINE\_LINUX\_DEFAULT_ line. now just update grub: `update-grub` and reboot your pc

---
{: data-content="backup and flash"}

to flash rom i will use flashrom installed from void repos, even though flashprog is preferred, but there are some lib problems on void linux with flashprog and i don't have time nor patience to deal with it right now

with flashrom installed, read your current rom and store it somewhere safe: 
```
flashrom -p internal:laptop=force_I_want_a_brick,boardmismatch=force -r dump.bin
```
do this **twice**, but change the name of the rom you read so you don't overwrite the first one. this is necessary so we can compare the outputs and confirm that these readings were executed without problems.

after the reading is done, compare the outputs: 
```
diff dump.bin dump2.bin
```
if this command outputs nothing, everything is okay. if the command states that those files differ, something is wrong and you should not continue flashing and instead should investigate the issue.

now go to your **injected rom** (libreboot.rom in my case) and write it to the chip:
```
flashrom -p internal:laptop=force_I_want_a_brick,boardmismatch=force -w libreboot.rom
```
after that *read* the chip again and then compare outputs with diff to make sure that the rom you just wrote matches the one you wanted to write to the chip:
```
flashrom -p internal:laptop=force_I_want_a_brick,boardmismatch=force -r dump3.bin
diff libreboot.rom dump3.bin
``` 

if roms match, everything is ok and you can reboot your pc. if they don't match or flashrom gave you some errors, you should not reboot your pc and instead investigate, maybe even try to reflash your old rom. if you reboot your pc after bad flash, you will brick your pc.

---
{: data-content="enable security"}

open _/etc/default/grub_ with a text editor and remove `iomem=relaxed` you added before. now update grub: `update-grub` and reboot.

---
layout: post
title:  "Changing background in libreboot GRUB rom"
date:   2024-10-12  15:59:00 +0200
categories: libreboot
---

This guides shows how to change GRUB background in already injected libreboot rom. I will use coreboot and grub tools built manually via lbmk. I will insert new grub config specifying custom background file, and then test it using _grubtest.cfg_ and after that finalize using just _grub.cfg_. At the end I also show how to edit grub config prior to building your roms and change background this way, instead of changing grub config by modifying injnected rom. I'm doing this process on Void Linux, and the rom I will be modyfing is for Lenovo Thinkpad T420, version of libreboot 20241008.

---
{: data-content="setting up lbmk"}

check for coreboot tree for your board and build it:
```
./mk -b coreboot list
./mk -d coreboot t420_8mb
```

we will use tool called _cbfstool_ located in _elf/cbfstool/default/cbfstool_ after you build coreboot tools

build default grub tree to get grub config:
```
./mk -b grub default
```

---
{: data-content="preparing background"}

some images work, some don't. you have to try them out to know. i recommend images with size 1280x800. also, the image should not be too big, because the space available in the rom is limited, i usually try not to exceed 250 kB. the image i will be using in this example is from libreboot leah's personal page, you can get it [here](https://av.vimuser.org/dankgnu_purple.png)

only thing you have to do with the downloaded image is to name it background.png:
```
mv images/dankgnu_purple.png images/background.png
```

---
{: data-content="editing grub config"}

copy example grub config (after you build grub tools) to any location where you will modify it. also rename it to _grubtest.cfg_:
```
cp config/grub/default/config/payload ../roms/new/t420/grub/grubtest.cfg
```

open it using your preferred text editor, and change this part:
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/build-grub-cfg-before.png" %}

to this:
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/build-grub-cfg-after.png" %}

---
{: data-content="modyfing rom"}

add background file to the rom:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom add -f ../roms/images/background.png -n background.png -t raw
```

add custom _grubtest.cfg_ to the rom:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom add -f ../roms/new/t420/grub/grubtest.cfg -n grubtest.cfg -t raw
```

print rom contents and verify that both background file and _grubtest.cfg_ are present:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom print
```

my output looks like this:
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/rom-print-output.png" %}

now you can flash your modified rom

---
{: data-content="testing grubtest.cfg"}

after flashing new rom, boot your pc and choose **Load test configuration...** in grub menu
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/boot-grubtest.png" %}

then you should see your new background
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/enjoy.png" %}

---
{: data-content="renaming grub config and adding it to the rom"}

**BE WARY, THIS MIGHT NOT WORK** - atleast it's not working for me, the _grub.cfg_ acts the same as _grubtest.cfg_, it does not load automatically and i have to load it from the grub menu. if this is the case for you, use the last section to build rom image from source with prior modified grub config that will be used in the build process

if everything works correctly, you can remove test configuration and insert final grub config, so it loads automatically

rename _grubtest.cfg_ to _grub.cfg_:
```
mv grubtest.cfg grub.cfg
```

remove _grubtest.cfg_ from rom:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom remove -n grubtest.cfg
```

insert renamed _grub.cfg_:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom add -f ../roms/new/t420/grub/grub.cfg -n grub.cfg -t raw
```

print rom contents and check:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/grub/libreboot.rom print
```

now you can flash modified rom again

---
{: data-content="grub.cfg not booting automatically? change the config in source and build it!"}

we will change the source config that is used for building images, so that background in CBFS takes precedence over background in MEMDISK. if there is not background in CBFS, the default one in MEMDISK should still be loaded

backup the config that was created by building grub tools, in case you want to revert to it
```
cp config/grub/default/config/payload config/grub/default/config/payload.bak
```

edit the original _payload_ file and swap memdisk and cbfsdisk values:
```
vim config/grub/default/config/payload
```

this is how it looks before:
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/build-grub-cfg-before.png" %}

this is how it looks after:
{% include zoomable_image.html img_link="/assets/images/2024-10-12-changing-background-in-grub-rom/build-grub-cfg-after.png" %}

after editing the config you can build your rom images from source:
```
./mk -b coreboot t420_8mb
```

copy new rom somewhere else and rename it to libreboot.rom:
```
cp bin/t420_8mb/seagrub_t420_8mb_libgfxinit_corebootfb_usqwerty.rom ../roms/new/t420/new3/libreboot.rom 
```

add background file to the rom:
```
./elf/cbfstool/default/cbfstool ../roms/new/t420/new3/libreboot.rom add -f ../roms/images/background.png -n background.png -t raw
```

and now just flash your image. new background should be loaded automatically

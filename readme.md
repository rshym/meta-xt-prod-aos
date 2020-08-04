# meta-xt-prod-aos

## Content
This document is long enough to has it's own content. So below you will find:
* What is meta-xt-prod-aos
* Quick start
  * Some basic and most used commands
* Usage and setup
  * Serial over USB
  * SSH over ethernet
  * What to do "on board"
* How to build
  * Few build hints
* How to install
  * Write to microSD card
  * Write to eMMC
* How to prepare board (loaders and u-boot settings)
* References

Important note.
All PC-related steps described bellow are intended to be run on Linux, and were tested on Ubuntu 16, 18.
Windows and MacOS are not supported, sorry.

## What is meta-xt-prod-aos

This is meta layer that allows you to build AOS running as guest OS under Xen hypervisor on some Renesas' R-CAR boards.
Taking into account that you are interested in sources of AOS, we suppose that you understand what previous sentence means.
In case you need additional info look into last section of this document - "References".
What AOS does?
* it runs some automotive telemetry services
* communicates with secure AOS cloud
* receives updates from cloud
* provides data from sensors to user's OS (like Andoid or AGL)
This is fault-tolerate hub for communications with cloud, automotive sensors and user-facing OS.
Why it is under Xen hypervisor?
* it can reboot promptly in case of some failure met
* it can share hardware with some other special OS


## Quick start

As far as meta-xt-prod-aos is meta layer, it's not intended to be build by itself, but as part of bigger system.
So, you need to use our tool to build required distibutive.
```
git clone git@github.com:xen-troops/build-scripts.git
cd build-scripts
python ./build_prod.py --machine h3ulcb --product aos \
    --config local_aos.cfg --with-local-conf --with-do-build
```
Build takes about three hours on iCore7 with 32GB and SSD.
You need to have 500GB of free space disk (disks) mentiond in [path] section of local_aos.cfg.

Go to `deploy` folder and run
```
./mk_sdcard_image.sh -p . -d ./image.img -c aos
```
You have image that can be flashed to microSD card using `dd`.
```
sudo dd if=./image.img of="your_sd_card_like_/dev/sdX" bs=1M status=progress
```
or bmaptool which is strongly recommended due to higher speed.
```
sudo bmaptool copy --bmap image.img.bmap ./imgage.img your_sd_card_like_/dev/sdX
```
But if you want to have fastest speed of running - use internal eMMC of StarterKit.
This step is a little bit more complex than `dd` or `bmaptool` so you need to follow to section "How to install".

### Some basic and most used commands
Inside Dom0
```
xl list                           - list running domains
xl console DomD                   - switch to domD's console
xl destroy DomF                   - destroy domF
xl create /xt/dom.cfg/domf.cfg -c - create domF and switch to it's console
```
Inside domD/domF
```
Ctrl+5                            - return to dom0 from domD/domF
```

## Usage and setup
AOS is not intented to use any display or keyboard so you need to connect to board by:
1) serial over USB
2) SSH over ethernet

### Serial over USB
You need to connect your PC to board by USB-microUSB cable.

Use your favorite serial console (minicom, cu etc).

Set port to: 115200, 8N1, software flow control.

Connect your PC to board's micro USB connector named "Debug serial".

If needed - use detailed and labeled photo of StarterKit at https://elinux.org/R-Car/Boards/H3SK

Turn on board and see messages.

### SSH over ethernet
At first determine IP that is assigned to board.

If serial already works - you can login to domD and check:
```
dom0 login: root
# xl console DomD
domd login:root
# ifconfig
```
Or check your DHCP server.

Connect to board
```
ssh root@<board_ip>
```
After that you will be logged straight into DomD.
Also you can connect to DomF, just use port 23:
```
ssh -p 23 root@<board_ip>
```

### What to do "on board"
Use `root` to login to dom0 or domD (yes, it's just demo, and yes, we will close this security hole).

Main program inside dom0 is `xl`. You can use
```
xl list                            ## to see list of active domains
xl console DomD                    ## to switch to DomD's console (press Ctrl-5 to return to Dom0)
xl destroy DomF
xl create -c /xt/dom.cfg/domf.cfg  ## to create domain using specified config
```
Detailed manual for `xl` you can find at https://xenbits.xen.org/docs/unstable/man/xl.1.html

DomD is quite regular Linux so you can use lots of regular Linux tools to do lots of regular Linux things.

Also pay attention that domD is home for backend drivers for domF. This means that domF is working with real hardware using drivers in domD. And if you stop network in domD, domF will be isolated from world as well.

If you want to return to dom0 - use `Ctrl-5`.

DomF has it's own special application and has no access to real hardware.


## How to build
Due to complex dependecies between different projects, you need to use special python script to build final product. Script will pull required layers and set configuration files.
So you need to get script from github and edit few lines in config.
Either clone repo
```
git clone https://github.com/xen-troops/build-scripts.git
```
or just download and unpack archive
```
wget https://github.com/xen-troops/build-scripts/archive/master.zip
```
Folder contains two python files (script itself), and configs for different products and specific cases.

Create `my_aos.cfg` with following content:
```
[git]
xt_history_uri = ""
xt_manifest_uri = git@github.com:xen-troops/meta-xt-products.git
[path]
workspace_base_dir = /<your_storage>/work_aos/build
workspace_storage_base_dir = /<your_storage>/storage
workspace_cache_base_dir = /<your_storage>/work_aos/tmp
[local_conf]
XT_GUESTS_INSTALL = "domf"
XT_GUESTS_BUILD = "domf"
```
Pay attention that you need to specify your own drive with about 500GB of free space.

Now you can run script:
```
python ./build_prod.py --machine h3ulcb --product aos --config my_aos.cfg --with-local-conf --with-do-build
```

### Few build hints
Pay attention that mentioned above command will erase `workspace_base_dir` and start clear build.

* If case of interruption you can just re-run script keeping all fetched sources and :
```
python ./build_prod.py --machine h3ulcb --product aos --config my_aos.cfg --continue-build
```

* You can get "essence" of prod-aos, apply some pathes/modifications and continue build:
```
python ./build_prod.py --machine h3ulcb --product aos --config my_aos.cfg --with-local-conf
pushd /<your_storage>/work_aos/build/meta-xt-prod-aos
# do something, apply patches
popd
python ./build_prod.py --machine h3ulcb --product aos --config my_aos.cfg --continue-build
```

* To clear build products and intermediate files:
```
rm -rf /<your_storage>/work_aos/tmp/
cd /<your_storage>/work_aos/build/build
# remove everything except conf/ folder
```

## How to install

### Write to microSD card


### Write to eMMC
TODO

## References
* AOS cloud - https://aoscloud.io/
* If you are not aware what is meta layer - you may read Yocto's documentation at https://www.yoctoproject.org/docs/
* If you want to understand what is hypervisor nad guest OS - check https://xenproject.org/help/documentation/
* To review Renesas boards - check https://www.renesas.com/eu/en/solutions/automotive/adas/solution-kits/r-car-starter-kit.html


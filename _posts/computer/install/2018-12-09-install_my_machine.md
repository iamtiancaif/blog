install my machine
==================

  

I spent all 2 weeks to fit my machine in siov......

  

desktop
=======

hibernation
-----------

**Suspend and hibernate**   

sudo apt-get install tuxonice-userui hibernate  

\# use  

hibernate-disk

  

  

drawing
-------

sudo apt-get install pinta  

  

  

ibus-table
----------

sudo apt-get install ibus-table  

sudo apt-get install ibus-table-wubi  

ibus-daemon &      # log out then login again, or the input method inside console won't work correctly.  

  

lantern
-------

\# lantern can not start

sudo apt-get install libappindicator3-1  

screen shot
-----------

sudo dpkg -i copyq\_2.6.1\_Debian\_7.0\_amd64.deb  

sudo dpkg -i deepin-scrot\_2.0-0deepin\_all.deb

\# goto root directory where deepin scrot installed.

cd /usr/share  

sudo patch -p0 < /data/usb/jxz/soft/install/jxz-deep.patch  

  

calendar
--------

**Lightweight calendar to display when clicking the clock**  /data/knowledge/computer/os/linux/openbox/tint2/Lightweight\_calendar\_to\_display\_when\_clicking\_the\_clock.html  

  

sudo apt-get install libunique-1.0-0 gsimplecal

  
  
\# edit tint2 configuration file  
clock\_lclick\_command = /opt/gsimplecal/gsimplecal  

  

  

teamviewer
----------

You need to dowload `teamviewer_11.0.53191_i386.deb`

Then run the following command :

       apt-get install -f
    

and execute the following command lines as an root:

    dpkg --add-architecture i386
    apt-get update
    

install

    dpkg -i teamviewer_11.0.53191_i386.deb

  

ebook
-----

sudo apt-get install calibre 

  

xmind
-----

sudo dpkg -i /data/svn-soft/install/xmind/xmind-linux-3.4.1.201401221918\_amd64.deb

sudo cp /data/svn-soft/install/xmind/net.xmind.verify\_3.4.1.201401221918.jar /usr/local/xmind/plugins/ 

gpasswd no-internet  

  

\# if we haven't initialize the group password, we'll encounter a error: "sg: failed to crypt password with previous salt: Invalid argument"

goagent
-------

### ImportError: No module named OpenSSL

/data/sync/cloud-knowledge/computer/os/linux/scrappy/No\_module\_named\_OpenSSL.html   

  

### global name 'SSLContext' is not defined

/data/sync/cloud-knowledge/computer/python/error\_global\_name\_SSLContext\_is\_not\_defined.html 

  

  

bluetooth
---------

  

1914  sudo apt-get install blueman  
1915  sudo apt-get install pulseaudio  
1916  sudo apt-get install pavucontrol  
1917  pavucontrol  
1918  sudo apt-get purge pavucontrol  
1919  sudo apt-get install pulseaudio pulseaudio-module-bluetooth pavucontrol bluez-firmware  
1920  sudo service bluetooth restart  
1921  killall pulseaudio  
1922  sudo killall pulseaudio  
1923  exit  
1924  cd /data/delete/  
1925  wget /usr/lib/python2.7/dist-packages/gevent/ssl.py  
1926  wget https://pypi.python.org/packages/12/dc/0b2e57823225de86f6e111a65d212c9e3b64847dddaa19691a6cb94b0b2e/gevent-1.1.1.tar.gz#md5=1532f5396ab4d07a231f1935483be7c3  
1927  sudo update-alternatives --config x-www-browser  
1928  history | grep python-setuptools  
1929  xxnet.sh  
1930  exit  
1931  cd /data/sync/cloud-knowledge/computer/python/  
1932  xtouch error\_global\_name\_SSLContext\_is\_not\_defined.html  
1933  exit  
1934  sudo apt-get install bluez-utils bluez-gnome bluez-alsa  
1935  exit  
1936  sudo rfkill list  
1937  exit  
1938  sudo -i  
1939  exit  
1940  sudo bluetoothctl  
1941  exit  
1942  sudo hcitool scan  
1943  cat .asoundrc  
1944  hciconfig  
1945  hciconfig hci0 piscan  
1946  sudo hciconfig hci0 piscan  
1947  sudo dbus-send --system --dest=org.bluez /org/bluez/hci0 org.bluez.Adapter.SetMode string:discoverable  
1948  sudo hciconfig hci0 piscan  
1949  hciconfig  
1950  sudo pulseaudio -k  
1951  sudo pulseaudio --start  
1952  sudo pactl load-module module-bluetooth-discover  
1953  sudo apt-get install pulseaudio-bluetooth-module  
1954  sudo apt-get install pulseaudio-bluetooth  
1955  sudo apt-get install pavucontrol  

sudo pactl load-module module-bluetooth-discover  

               module bluetooth discover initialization failed  

  

it did not work so far.

  

  

### but after i performed these below, it worked. So strangebut after i performed these below, it worked. So strange

**PulseAudio can not load bluetooth module** /data/knowledge/computer/os/linux/debian/scrap/PulseAudio\_can\_not\_load\_bluetooth\_module.html  

sudo apt-get install pulseaudio-bluetooth-module  

sudo pactl load-module module-bluetooth-discover  

  

because some guy said that maybe the module was not installed. (in this thread: https://github.com/blueman-project/blueman/issues/158) I tried, and make it work!!!!!!! T-T

  

flash plugin chromium
---------------------

more detail under directory /data/knowledge/computer/os/linux/debian/scrap/flash.

  

  

**Can not install Flash Player on Debian Jessie**  /data/knowledge/computer/os/linux/debian/scrap/Can\_not\_install\_Flash\_Player\_on\_Debian\_Jessie.html  

**not work**.

  

### not work

sudo apt-get install pepperflashplugin-nonfree  
sudo update-pepperflashplugin-nonfree --install  

  

  

### Getting-Flash

https://wiki.ubuntu.com/Chromium/Getting-Flash  

As of 2015-05, the old "pepperflashplugin-nonfree" is deprecated in favor of an official, maintained, one-step package called **adobe-flashplugin**, which works for Firefox and Chromium and derivatives, but not for Yandex.Browser 46.0.2490.3623 beta (64-bit version) on November 2015. No terminals, no multiverse.

Add the Canonical Partners software source, and install _adobe-flashplugin_. See [Chromium/Getting-Partner-Flash](https://wiki.ubuntu.com/Chromium/Getting-Partner-Flash).

  

  

#### jxz

I put it away at /data/usb/jxz/soft/install/flash\_player\_ppapi\_linux.x86\_64.tar.gz

  

### jxz work for firefox

https://wiki.debian.org/PepperFlashPlayer/Installing  

#### Debian 8 "Jessie"

Pepper Flash Player is available in the contrib section of jessie-backports. To make jessie-backports packages available on your system, add a line like this to /etc/apt/sources.list or /etc/apt/sources.list.d/backports.list:

deb http://http.debian.net/debian jessie-backports main contrib

After adding the repository to your sources, update the local index:

apt-get update

See the [Backports](https://wiki.debian.org/Backports) page and [https://backports.debian.org/](https://backports.debian.org/) for more information.

#### bits / amd64

*   aptitude install pepperflashplugin-nonfree browser-plugin-freshplayer-pepperflash
    

  

#### jxz

update-alternatives: using /usr/lib/browser-plugin-freshplayer-pepperflash/libfreshwrapper-flashplayer.so to provide /usr/lib/mozilla/plugins/flash-mozilla.so (flash-mozilla.so) in auto mode  

  

**for firefox**:

ln -s /usr/lib/browser-plugin-freshplayer-pepperflash/libfreshwrapper-flashplayer.so /usr/lib/mozilla/plugins/flash-mozilla.so

  

### jxz work for chromium

  

cp libpepflashplayer.so /usr/lib/pepperflashplugin-nonfree/

  

  

jekyll
------

### debian 8

sudo apt-get -y install curl git-core nodejs  
chmod +x setup\_8.x 
sudo apt-get install nodejs  
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison nodejs subversion  
wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.3.tar.gz  
tar -zxf ruby-2.5.3.tar.gz   
cd ruby-2.5.3/  
ls  
./configure   
make  
sudo make install  
sudo gem install github-pages  

  

refer to **How To Install Ruby on Rails on Ubuntu 12.04 from Source**


### debian 9

sudo apt-get -y install curl git-core nodejs  
sudo apt-get -y install jekyll  
  

I put these files in /data/usb/jxz/installer/install/ruby

vmware
------

cd /usr/lib/vmware/modules/source/

cp /data/knowledge/computer/os/linux/topic/hibernate/vm-patch/\* ./

  

  

file manager
------------

xdg-mime default nautilus.desktop inode/directory application/x-gnome-saved-search  

sudo apt-get install exo-utils  

exo-preferred-applications  

  

  

sound recorder
--------------

sudo apt-get install gnome-sound-recorder  

  

  

gwrite
------

  

Traceback (most recent call last):  
  File "/usr/bin/gwrite", line 16, in <module>  
    gwrite.gwrite.main()  
  File "/usr/lib/pymodules/python2.7/gwrite/gwrite.py", line 2459, in main  
    s.bind(ctlfile)  
  File "/usr/lib/python2.7/socket.py", line 224, in meth  
    return getattr(self.\_sock,name)(\*args)  
socket.error: \[Errno 2\] No such file or directory  

  

  

  

**jxz**: 

When I ran gwrite as normal user these error occurred but root was ok.

Seems it want to setup some connection, so I just commentted out the lines binding sockets in file **gwrite.py** around line 2459.

  

compile kernel
==============

\# install dependencies

sudo apt-get install libssl-dev build-essential ncurses-dev xz-utils kernel-package  

  

\# for hibernate,   just make the situation better, not resolve the problem.

sudo apt-get install pm-utils

  

  

# /data/knowledge/computer/os/linux/scrappy/Suspend\_and\_hibernate.html

\# don't use hibernate-disk

sudo apt-get install tuxonice-userui hibernate   

  

  

sudo sysctl vm.swappiness=1

  

  

vim /etc/sysctl.conf  

vm.swappiness = 1  

  

  

\# modify the grub parameters

\# vim /etc/default/grub

GRUB\_DEFAULT=0  
GRUB\_TIMEOUT=5  
GRUB\_DISTRIBUTOR=\`lsb\_release -i -s 2> /dev/null || echo Debian\`  
GRUB\_CMDLINE\_LINUX\_DEFAULT="quiet"

\# GRUB\_CMDLINE\_LINUX="initrd=/install/initrd.gz"  
**GRUB\_CMDLINE\_LINUX=""**  

  

developing
==========

  

performance
-----------

sudo apt-get install sysstat  

  

svn
---

\# sudo wget -O - http://opensource.wandisco.com/wandisco-debian.gpg | sudo apt-key add -  

cat /data/usb/jxz/soft/software/licenses/wandisco-debian.gpg | sudo apt-key add -  

deb http://opensource.wandisco.com/debian jessie svn19  

sudo apt-get update

sudo apt-get install subversion

  

l2tp client
-----------

sudo apt-get install xl2tpd   

sudo dpkg -i /data/svn-soft/install/openswan\_2.6.37-3+deb7u1\_amd64.deb 

  

vim
---

i have built one deb package under /data/usb/jxz/soft/install.

  

so.

sudo apt-get remove vim-tiny vim-common vim-gui-common vim-nox  

sudo  vim/vim\_20170223-1\_amd64.deb

### Building Vim from source

  

/data/knowledge/computer/os/linux/vim/topic/ide\_like/Building\_Vim\_from\_source.html  

  

### Help Maintain Vundle

/data/knowledge/computer/os/linux/vim/topic/ide\_like/Help\_Maintain\_Vundle.html  

  

YouCompleteMe  

cd ~/.vim/bundle/YouCompleteMe

. /data/sync/software/script/terminal-proxy.sh

git submodule update --init --recursive

./install.py --clang-completer

./install.py --all

  

  

### install ctags

sudo apt-get install exuberant-ctags  

  

  

### install taglist

/data/knowledge/computer/os/linux/vim/topic/ide\_like/install\_taglist.html  

  

  

### install cscope

sudo apt-get install cscope  

  

/data/knowledge/computer/os/linux/vim/topic/ide\_like/Browsing\_Source\_Code\_in\_Linux\_---\_Vim\_Cscope.html  

  

  

StarUML
-------

install\_StarUML.html  

crack\_staruml.html  

  

  

sudo dpkg -i libgcrypt11\_1.5.3-2ubuntu4.2\_amd64.deb  
sudo dpkg -i StarUML-v2.8.0-64-bit.deb  

  

### crack

/opt/staruml/www/license/node/LicenseManagerDomain.js

  

>     function validate(PK, name, product, licenseKey) {  
>         var pk, decrypted;  
>         // edit by 0xcb  
>         return {  
>             name: "0xcb",  
>             product: "StarUML",  
>             licenseType: "vip",  
>             quantity: "mergades.com",  
>             licenseKey: "later equals never!"  
>         };  
>         try {  
>             pk = new NodeRSA(PK);  
>             decrypted = pk.decrypt(licenseKey, 'utf8');  
>         } catch (err) {  
>             return false;  
>         }  
>         var terms = decrypted.trim().split("\\n");  
>         if (terms\[0\] === name && terms\[1\] === product) {  
>             return {   
>                 name: name,   
>                 product: product,   
>                 licenseType: terms\[2\],  
>                 quantity: terms\[3\],  
>                 licenseKey: licenseKey  
>             };  
>         } else {  
>             return false;  
>         }  
>     }
---
author: jeff
comments: true
date: 2012-04-24 18:39:35+00:00
layout: post
title: "nanoBSD et soekris"
categories: FreeBSD nanoBSD
tags: FreeBSD nanoBSD
---


Il y a quelques années de cela j'ai acheté une soekris.  
Il s'agit d'une carte mère servant principalement à la mise en place d'appliance réseau. Pour plus d'infos voir [ici](http://www.soekris.com).  

Je possède une net4801 qui, dans les grandes lignes, est équipée de :

  * un processeur NSC de 233 Mhz
  * 128 Mo de RAM
  * 3 ports RJ45
  * 1 port miniPCI
  * 1 port PCI sous 3,3V
  * 1 port UDMA33 2,5"
  * 1 port USB 1 (wouaï ça date ça dites donc !)
  * 1 port CompactFlash

Bref, comparé à ce qu'on trouve aujourd'hui ce n'est pas folichon.  
En tout cas elle m'a servi bien des années comme passerelle wifi sous Debian (adaptée aux petits oignons) puis plus tard sous pfSense. Mais depuis 2-3 ans elle dort bien rangée dans une boite. N'aimant pas la voir moisir dans son coin, je me suis dit qu'il serait intéressant de la remettre en service d'autant plus que cela me permettrait de bosser tranquillement l'administration système sous GNU/Linux.  
Me voilà donc parti.


**Question numéro 1 : quel OS installer ?**
  
[NanoBSD](http://www.freebsd.org/doc/en/articles/nanobsd/). Il s'agit d'un script disponible dans l'arbre des sources de FreeBSD, permettant de construire une image disque d'un système BSD monté en mémoire.
Le principe est simple : il faut installer une FreeBSD avec un arbre des sources à jour, mettre en place les fichiers de configurations de NanoBSD (du noyau, des ports à installer, des fichiers à intégrer dans l'image finale), lancer le script, attendre que ce soit chaud et enfin copier l'image sur un support à mémoire flash.


**Question numéro 2 : komenkonfé ?**
  
D'abord s'installer une FreeBSD. Pour ma part, je travaille à l'aide d'une machine virtuelle.  
Ensuite, mettre à jour la FreeBSD à la dernière version si nécessaire -> [Mettre à jour](http://www.mala.fr/mise-a-jour/).  
Enfin, mettre à jour l'arbre des ports -> [Commandes de post-installation](http://www.mala.fr/commandes-de-post-installation/).  
Maintenant il s'agit d'éditer le fichier de configuration à envoyer au script nanobsd.sh.


**Question numéro 3 : le script nanobsd.sh ?!**
  
Il se trouve là dedans : `/usr/src/tools/tools/nanobsd/`.

    # cd /usr/src/tools/tools/nanobsd
    # sh nanobsd.sh -h
    Usage: nanobsd.sh [-bfiknqvw] [-c config_file]
            -b      suppress builds (both kernel and world)
            -f      suppress code slice extraction
            -i      suppress disk image build
            -k      suppress buildkernel
            -n      add -DNO_CLEAN to buildworld, buildkernel, etc
            -q      make output more quiet
            -v      make output more verbose
            -w      suppress buildworld
            -c      specify config file


**Question numéro 4 : la configuration et la compilation ?**  
  
Rapide présentation des fichiers nécessaire pour lancer la compilation.

* Le fichier `NET4801.conf`.  
C'est le fichier de configuration à passer au script pour personnaliser l'image finale : nom de l'image, taille en fonction de celle du support, ports à inclure, paramétrages du noyau, etc...

* Viens ensuite le fichier `FlashDevice.sub`.  
Ce fichier contient les tailles des images à réaliser en fonction des marques et modèles de supports de stockage.
On peut personnaliser ce fichier.

* Le dossier `Files`.  
Contient une arborescence de fichiers/dossiers qui seront ajoutés à l'image finale.

* Le fichier `NET4801`.  
Fichier de configuration personnalisé du noyau, à mettre dans `/usr/src/sys/i386/conf/`.
  

On peut enfin lancer la compilation :  
    
    # sh nanobsd.sh -c NET4801.conf

On obtient une image .img que l'on peut tester sous qemu avec :
    
    # qemu-system-i386 -hdb BSDg_130513-2327.img -m 128 -no-acpi\
     -nographic -cpu pentium -net user,hostfwd=::5555-:80\
     -net nic,model=rtl8139
    
Cette commande configure le réseau virtuel avec le pilote RealTek 8139 (il faut penser à la compiler dans le noyau de NanoBSD), redirige le port 5555 de l'hôte vers le port 80 de l'invité (notre système NanoBSD).  

Voici maintenant le contenu de mon fichiers de configuration de NanoBSD.  
Le répertoire de compilation est nommé en fonction de la date et l'heure de lancement du script et il est créé dans un point de montage avec beaucoup de place.  
L'image est compilée pour une carte CompactFlash de 8 Go.
     
    #!/bin/sh
    
    ################################################################
    #                                                              #
    # NanoBSD Config file for Soekris NET4801                      #
    #                                                              #
    ################################################################
    
    set -e
    
    # Name of this NanoBSD build.  (Used to construct workdir names)
    NANO_NAME=net4801_$(date +%d%m%y-%H%M)
    
    # Source tree directory
    NANO_SRC=/usr/src
    
    # Where nanobsd additional files live under the source tree
    NANO_TOOLS=tools/tools/nanobsd
    
    # Parallel Make
    NANO_PMAKE="make -j 2"
    
    # Target arch
    NANO_ARCH="i386"
    
    NANO_OBJ="/mnt/obj/$(date +%d%m%y)"
    
    NANO_BOOTLOADER="boot/boot0"
    
    # Options to put in make.conf during buildworld only
    CONF_BUILD='
    #WITHOUT_KLDLOAD=YES
    #WITHOUT_PAM=YES
    '
    
    # Options to put in make.conf during installworld only
    CONF_INSTALL='
    WITHOUT_ACPI=YES
    WITHOUT_BLUETOOTH=YES
    WITHOUT_CVS=YES
    WITHOUT_FORTRAN=YES
    WITHOUT_HTML=YES
    WITHOUT_LPR=YES
    WITHOUT_MAN=YES
    #WITHOUT_SENDMAIL=YES
    WITHOUT_SHAREDOCS=YES
    WITHOUT_EXAMPLES=YES
    WITHOUT_CALENDAR=YES
    WITHOUT_MISC=YES
    WITHOUT_SHARE=YES
    '
    
    # Options to put in make.conf during both build- & installworld.
    CONF_WORLD='
    CPUTYPE?=i386
    CFLAGS=-O2 -fno-strict-aliasing -pipe
    #WITHOUT_BIND=YES
    WITHOUT_MODULES=YES
    #WITHOUT_KERBEROS=YES
    WITHOUT_GAMES=YES
    WITHOUT_RESCUE=YES
    #WITHOUT_LOCALES=YES
    WITHOUT_SYSCONS=YES
    #WITHOUT_INFO=YES
    '
    
    # Kernel config file to use
    NANO_KERNEL=NET4801-1
    
    # Customize commands.
    NANO_CUSTOMIZE=""
    
    # Newfs paramters to use
    NANO_NEWFS="-b 4096 -f 512 -i 8192 -O2 -m 0"
    
    # The drive name of the media at runtime
    NANO_DRIVE=ad1
    
    # Size of the /etc ramdisk in 512 bytes sectors
    NANO_RAM_ETCSIZE=$((1024/512*1024*10))		# 10Mo
    
    # Size of the /tmp+/var ramdisk in 512 bytes sectors
    NANO_RAM_TMPVARSIZE=$((1024/512*1024*10))	# 10Mo
    
    # Number of code images on media (1 or 2)
    NANO_IMAGES=1
    
    NANO_IMGNAME="BSDg_$(date +%d%m%y-%H%M).img"
    
    # 0 -> Leave second image all zeroes so it compresses better.
    # 1 -> Initialize second image with a copy of the first
    NANO_INIT_IMG2=0
    
    # Size of code file system in 512 bytes sectors
    # If zero, size will be as large as possible.
    NANO_CODESIZE=0 #$((1024/512*1024*768)) 	# 768Mo
    
    # Size of configuration file system in 512 bytes sectors
    # Cannot be zero.
    NANO_CONFSIZE=$((1024/512*1024*64)) 		# 64Mo
    
    NANO_DATASIZE=$((1024/512*1024*1024))		# 1024Mo = 1Go
    
    # Backing type of md(4) device
    # Can be "file" or "swap"
    NANO_MD_BACKING="file"
    
    # Label name
    # Alphacharacter only
    NANO_GLABEL_SYS="SYS"
    NANO_GLABEL_CFG="CFG"
    NANO_GLABEL_DATA="DATA"
    
    
    # Media definition from FlashDevice.sub
    FlashDevice patriot 8g 
    #FlashDevice pqi 512mb
    #FlashDevice pny 512mb
    
    
    ### ADD PACKAGES ### 
    
    # Use your prefered mirror or use the remote fetching feature of pkg_add
    # If you have smaller CF build your own packages with custom knobs
    # and see cust_pkg() from nanobsd.sh
    add_pkg() {
        pkg=`echo $1 | sed -e 's/-/_/'`
        eval "
        custom_add_pkg_${pkg} () {
            chroot \${NANO_WORLDDIR} /bin/sh -exc \
            #'pkg_add -v ftp://ftp.fr.freebsd.org/pub/FreeBSD/ports/i386/packages-8.0-release/Latest/$1.tbz'
            'pkg_add -r $1'
        }
        customize_cmd custom_add_pkg_${pkg}
        "
    }
    
    ### ADD PORTS ### 
    
    add_port () {
            port=`echo $1 | sed -e 's/\//_/'`
            eval "
            add_port_${port} () {
                    mkdir -p \${NANO_WORLDDIR}/usr/ports
                    mount -t unionfs -o noatime /usr/src \
                            \${NANO_WORLDDIR}/usr/src
                    mount -t unionfs -o noatime /usr/ports \
                            \${NANO_WORLDDIR}/usr/ports
                    mkdir -p \${NANO_WORLDDIR}/dev
                    mount -t devfs devfs \${NANO_WORLDDIR}/dev
                    mkdir -p \${NANO_WORLDDIR}/usr/pobj
                    mkdir -p \${NANO_WORLDDIR}/usr/workdir
                    cp /etc/resolv.conf \${NANO_WORLDDIR}/etc/resolv.conf
                    chroot \${NANO_WORLDDIR} /bin/sh -exc \
                            'make WRKDIRPREFIX=/usr/workdir -C /usr/ports/$1 \
                            install BATCH=yes $2'
                    rm \${NANO_WORLDDIR}/etc/resolv.conf
                    rm -rf \${NANO_WORLDDIR}/usr/obj
                    rm -rf \${NANO_WORLDDIR}/usr/pobj
                    rm -rf \${NANO_WORLDDIR}/usr/workdir
                    umount \${NANO_WORLDDIR}/dev
                    umount \${NANO_WORLDDIR}/usr/ports
                    umount \${NANO_WORLDDIR}/usr/src
                    rmdir \${NANO_WORLDDIR}/usr/ports
            }
            customize_cmd add_port_${port}
            "
    }
    
    ### Port list
    
    # devel
    add_port "devel/libdlmalloc"
    add_port "devel/libtool"
    
    # dns
    #add_port "dns/libidn"
    #add_port "dns/dnsmasq"
    add_port "dns/djbdns" "-DWITH_IPV6 -DWITH_SRV"
    add_port "dns/ddclient"
    
    # editors
    #add_port "editors/vim-lite" "-DWITHOUT_X11"
    add_port "editors/nano" "-DWITHOUT_X11"
    
    # ftp
    add_port "ftp/pure-ftpd" "-WITH_LANG=french -DWITH_LDAP -DWITH_MYSQL -DWITH_PGSQL -DWITH_PRIVSEP -DWITH_TLS -DWITH_PERUSERLIMITS -DWITH_UPLOADSCRIPT -DWITH_UTF8 -DWITH_SENDFILE -DWITH_LARGEFILE -DWITH_VIRTUALCHROOT"
    
    # lang
    #add_port "lang/perl5.10" "WITHOUT_PERL_MALLOC=yes" # not installed because perl5.10 is a dependency of freeradius2 port 
    
    # net
    #add_port "net/nload"
    #add_port "net/mtr" "-DWITHOUT_X11"
    #add_port "net/ntraceroute"
    #add_port "net/freeradius2"
    #add_port "net/openldap24-server"
    add_port "net/rsync" "-DWITH_POPT_PORT -DWITH_ICONV"
    #add_port "net/miniupnpd"
    add_port "net/mpd5"
    add_port "net/isc-dhcp42-server"
    
    #add_port "databases/rrdtool12"
    add_port "net-mgmt/net-snmp"
    add_port "net-mgmt/darkstat"
    
    # security
    add_port "security/nmap"
    #add_port "security/openvpn"
    add_port "security/stunnel"
    
    # shells
    add_port "shells/zsh"
    
    # sysutils
    add_port "sysutils/pstree"
    add_port "sysutils/screen" "-DWITHOUT_INFO"
    add_port "sysutils/lsof"
    add_port "sysutils/dmidecode"
    add_port "sysutils/freecolor"
    add_port "sysutils/tmux"
    
    # www
    add_port "www/lighttpd"
    add_port "www/py-flup"
    #add_port "www/squid31" "-DWITHOUT_KERBEROS -DWITH_SQUID_LDAP_AUTH -DWITH_SQUID_SASL_AUTH -DWITH_SQUID_SSL -DWITH_SQUID_PINGER -DWITH_SQUID_WCCPV2  -DWITH_SQUID_ARP_ACL -DWITH_SQUID_PF -DWITH_SQUID_VIA_DB"
    #add_port "www/squidguard"
    #add_port "www/squid_radius_auth"
     
    
    ### CUSTOMIZATION ###
    
    custom_misc() {
        # quick boot
        #echo "autoboot_delay=\"0\"" >> ${NANO_WORLDDIR}/boot/loader.conf
        #echo "beastie_disable=\"YES\"" >> ${NANO_WORLDDIR}/boot/loader.conf
    
        # Enable serial console
        #echo " -h" >> ${NANO_WORLDDIR}/boot.config
        #sed -i "" -e '/^ttyu0/s/ std.*\".*off / al\.9600\" vt100 on /' ${NANO_WORLDDIR}/etc/ttys
        #sed -i "" -e '/^ttyv[0-8]/s/on /off /' ${NANO_WORLDDIR}/etc/ttys
    
        # Disable acpi
        echo "hint.acpi.0.disabled=\"1\"" >> ${NANO_WORLDDIR}/boot/device.hints
    
        # Disable sendmail daemon and relay via mail.mala.fr
        #echo "sendmail_enable=\"NONE\"" >> ${NANO_WORLDDIR}/etc/rc.conf
        #sed -i "" -e "s/^D{MTAHost}.*$/D{MTAHost}[mail.mala.fr]/" ${NANO_WORLDDIR}/etc/mail/submit.cf
    
        # Redirect root mails to [YOUR_MAIL_ADDRESS]
        sed -i "" -e 's/me@my.domain$/[YOUR_MAIL_ADDRESS]/ ; s/^#root:/root:/' /etc/aliases
        newaliases
    }
    
    custom_chpass () {
        chroot ${NANO_WORLDDIR} chpass -s /usr/local/bin/zsh root
    }
    
    cleanup_dot_a () {
        find ${NANO_WORLDDIR} -name '*.a' -ls -delete
    }
    
    custom_kernelgzip () {
        gzip -v9 ${NANO_WORLDDIR}/boot/kernel/kernel
    }
    
    post_custom_misc() {
        # Disable a warning at boot
        mkdir -p ${NANO_WORLDDIR}/usr/lib/aout
    }
    
    customize_cmd cust_install_files
    customize_cmd cust_allow_ssh_root
    customize_cmd custom_misc
    customize_cmd custom_kernelgzip
    customize_cmd cleanup_dot_a
    customize_cmd custom_chpass
    
    late_customize_cmd post_custom_misc
    

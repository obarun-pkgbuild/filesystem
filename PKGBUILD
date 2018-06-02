# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/filesystem
# 						Maintainer: Tom Gundersen <teg@jklm.no>


pkgname=filesystem
pkgver=2018.4
pkgrel=1
pkgdesc='Base Obarun filesystem'
arch=(x86_64)
license=('GPL')
url='https://www.obarun.org'
groups=('base')
#install='filesystem.install'
depends=('iana-etc' 'applysys')
backup=('etc/crypttab' 'etc/fstab' 'etc/group' 'etc/gshadow' 'etc/host.conf'
        'etc/hosts' 'etc/issue' 'etc/ld.so.conf' 'etc/motd' 'etc/nsswitch.conf'
        'etc/passwd' 'etc/profile' 'etc/resolv.conf' 'etc/securetty'
        'etc/shadow' 'etc/shells')
source=('crypttab' 'fstab' 'group' 'gshadow' 'host.conf' 'hosts'
        'issue' 'ld.so.conf' 'locale.sh' 'motd' 'nsswitch.conf' 'os-release'
        'passwd' 'profile' 'resolv.conf' 'securetty' 'shadow' 'shells'
        'sysusers' 'tmpfiles' 'modprobe.d.usb-load-ehci-first')
md5sums=('0672783048ee312d2fc768402fdb3488'
         '693c97f2c9a519bb97a17008e92c2b74'
         '26a96329a5523e5c11c50be58e6758c8'
         '822b75f0faca19a9c4cee334c63ab1b3'
         '7c944ff2ac3b4fc5e3c2f3e9aa1ed802'
         '4c8ed1804d6ff26714530dd43341b7a1'
         '4d378eac859ef2ed1520971b91ace639'
         '5deb9f890a4d08a245e9752ede77271e'
         'a9cabd3090a240bafbb05ce443d991df'
         'd41d8cd98f00b204e9800998ecf8427e'
         '9d858853fd10bb178dd8d09553302a7c'
         '4b1ede9239a8f606090ad2c8462771e2'
         'd49e3834aa82c71f71f058f23467bd6f'
         'cf6d37fa25ed367fc85b4a04e868352e'
         '1e025adcb3b94399bc74858440453501'
         'f04bcb2803afc4dcb95670fe87343b4d'
         '1e867e07ad9a04f40fa9e5e4aa1f1624'
         '6e0cf9e2ec97a424dfb11d41d2ef3c91'
         '973402ebe4c81b05628a30de3e393084'
         '0267a3a463f35eec8a31f40a720dfd86'
         'a8a962370cd0128465d514e6a1f74130')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

package() {
	cd "$pkgdir"

	#
	# setup root filesystem
	#
	for d in boot dev etc home mnt usr var opt srv/http run; do
		install -d -m755 $d
	done
	install -d -m555 proc
	install -d -m555 sys
	install -d -m0750 root
	install -d -m1777 tmp
	# vsftpd won't run with write perms on /srv/ftp
	# ftp (uid 14/gid 11)
	install -d -m555 -g 11 srv/ftp

	# setup /etc and /usr/share/factory/etc
	install -d etc/{ld.so.conf.d,skel,profile.d} usr/share/factory/etc
	
	for f in fstab group host.conf hosts issue ld.so.conf motd nsswitch.conf passwd resolv.conf securetty shells profile; do
			install -m644 "$srcdir"/$f etc/
			install -m644 "$srcdir"/$f usr/share/factory/etc/
	done
	ln -s ../proc/self/mounts etc/mtab
	for f in gshadow shadow crypttab; do
		install -m600 "$srcdir"/$f etc/
		install -m600 "$srcdir"/$f usr/share/factory/etc/
	done
	install -D -m644 "$srcdir"/modprobe.d.usb-load-ehci-first usr/lib/modprobe.d/usb-load-ehci-first.conf
	
	#touch etc/obarun-release
	install -m755 "$srcdir"/locale.sh etc/profile.d/locale.sh
	install -Dm644 "$srcdir"/os-release usr/lib/os-release

	# setup /var
	for d in cache local opt log/old lib/misc empty; do
		install -d -m755 var/$d
	done
	install -d -m1777 var/{tmp,spool/mail}

	# allow setgid games to write scores
	install -d -m775 -g 50 var/games
	ln -s spool/mail var/mail
	ln -s ../run var/run
	ln -s ../run/lock var/lock

	#
	# setup /usr hierarchy
	#
	for d in bin include lib share/misc src; do
		install -d -m755 usr/$d
	done
	for d in {1..8}; do
		install -d -m755 usr/share/man/man$d
	done

	#
	# add lib symlinks
	#
	ln -s usr/lib lib
	ln -s usr/lib lib64
	ln -s lib usr/lib64
	

	#
	# add bin symlinks
	#
	ln -s usr/bin bin
	ln -s usr/bin sbin
	ln -s bin usr/sbin

	#
	# setup /usr/local hierarchy
	#
	for d in bin etc games include lib man sbin share src; do
		install -d -m755 usr/local/$d
	done
	ln -s ../man usr/local/share/man
	ln -s bin usr/local/sbin
	
	# setup systemd-sysusers
	install -D -m644 "$srcdir"/sysusers usr/lib/sysusers.d/obarun.conf

	# setup systemd-tmpfiles
	install -D -m644 "$srcdir"/tmpfiles usr/lib/tmpfiles.d/obarun.conf
}

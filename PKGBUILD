# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/filesystem
# 						Maintainer: Tom Gundersen <teg@jklm.no>
# this pkgbuild was modified to eradicate systemd onto the base system
# and to configure os-release for obarun

pkgname=filesystem
pkgver=2017.03
pkgrel=3
pkgdesc='Base Obarun filesystem'
arch=(x86_64)
license=('GPL')
url='https://www.obarun.org'
groups=('base')
install='filesystem.install'
makedepends=('asciidoc')
depends=('iana-etc')
provides=('filesystem=$pkgver')
backup=('etc/fstab' 'etc/crypttab' 'etc/group' 'etc/hosts' 'etc/ld.so.conf' 'etc/passwd'
        'etc/shadow' 'etc/gshadow' 'etc/resolv.conf' 'etc/motd' 'etc/nsswitch.conf'
        'etc/shells' 'etc/host.conf' 'etc/securetty' 'etc/profile' 'etc/issue')
source=('group' 'issue' 'nsswitch.conf' 'securetty' 'host.conf' 'ld.so.conf'
        'passwd' 'shadow' 'fstab' 'crypttab' 'hosts' 'motd' 'os-release' 'resolv.conf'
        'shells' 'gshadow' 'profile' 'modprobe.d.usb-load-ehci-first'
        'locale.sh')
md5sums=('f72d348e6d890c02b9810256c7cddb7a'
         '4d378eac859ef2ed1520971b91ace639'
         '9e4533df61f0c82d6b2e2371f7376282'
         '4c4540eeb748bf1f71d631b8c1dcf0b3'
         'f28150d4c0b22a017be51b9f7f9977ed'
         '6e488ffecc8ba142c0cf7e2d7aeb832e'
         '34af3c7f09a6617435e94e95b0eb961e'
         '65d4c1ebd0d650e1f564861e8ec3eaa4'
         '693c97f2c9a519bb97a17008e92c2b74'
         'f0a5071f50d8864d2810c44e23eb00cc'
         '7bc65f234dfb6abf24e7c3b03e86f4ff'
         'd41d8cd98f00b204e9800998ecf8427e'
         '035f5dd5fbafe454989ab4b1bedfbc8a'
         '6f48288b6fcaf0065fcb7b0e525413e0'
         '22518e922891f9359f971f4f5b4e793c'
         '61a437db948f4e988b13cfeb1a86b6dc'
         '6c11d5af3bf8c770766e77312e7bd07f'
         'a8a962370cd0128465d514e6a1f74130'
         '71ed98c52e11ada1f936ac8cb14eecd9')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

lint() {
	# ensure that passwd is sync'd to shadow and group is sync'd to gshadow.
	local r=0

	local passwd shadow group gshadow

	for f in passwd shadow group gshadow; do
		mapfile -t "$f" < <(cut -d: -f1 "$f" | sort)
	done

	# we can cheat and do simple string comparison only because we can make some
	# assumptions about the data in these files
	if [[ ${passwd[*]} != "${shadow[*]}" ]]; then
		error 'passwd is not in sync with shadow!'
		r=1
	fi

	if [[ ${group[*]} != "${gshadow[*]}" ]]; then
		error 'group is not in sync with gshadow!'
		r=1
	fi

	return $r
}

build() {
	cd "$srcdir"

	lint

	#a2x -d manpage -f manpage archlinux.7.txt
}

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
	install -d -m555 -g ftp srv/ftp

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
	touch etc/obarun-release
	install -D -m644 "$srcdir"/modprobe.d.usb-load-ehci-first usr/lib/modprobe.d/usb-load-ehci-first.conf
	install -m755 "$srcdir"/locale.sh etc/profile.d/locale.sh
	install -Dm644 "$srcdir"/os-release "$pkgdir"/usr/lib/os-release

	# setup /var
	for d in cache local opt log/old lib/misc empty; do
		install -d -m755 var/$d
	done
	install -d -m1777 var/{tmp,spool/mail}

	# allow setgid games to write scores
	install -d -m775 -g games var/games
	ln -s spool/mail var/mail
	ln -s ../run var/run
	ln -s ../run/lock var/lock

	#
	# setup /usr hierarchy
	#
	for d in bin include lib share/misc src; do
		install -d -m755 usr/$d
	done
	for d in $(seq 8); do
		install -d -m755 usr/share/man/man$d
	done

	#
	# add lib symlinks
	#
	ln -s usr/lib "$pkgdir"/lib
	[[ $CARCH = 'x86_64' ]] && (
		ln -s usr/lib "$pkgdir"/lib64
		ln -s lib "$pkgdir"/usr/lib64
	)

	#
	# add bin symlinks
	#
	ln -s usr/bin "$pkgdir"/bin
	ln -s usr/bin "$pkgdir"/sbin
	ln -s bin "$pkgdir"/usr/sbin

	#
	# install archlinux(7) manpage
	#
	#install -D -m644 "$srcdir"/archlinux.7 usr/share/man/man7/archlinux.7

	#
	# setup /usr/local hierarchy
	#
	for d in bin etc games include lib man share src; do
		install -d -m755 usr/local/$d
	done
	ln -s ../man usr/local/share/man
	ln -s bin usr/local/sbin
}

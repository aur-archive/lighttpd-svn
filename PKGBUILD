# Maintainer: PyroPeter <pyropeter PLUS aur AT pyropeter DOT eu>

pkgname=lighttpd-svn
pkgver=0
pkgrel=2
license=('custom')
conflicts=('lighttpd')
pkgdesc="a secure, fast, compliant and very flexible web-server"
arch=('i686' 'x86_64')
url="http://www.lighttpd.net/"
depends=('pcre' 'openssl' 'zlib' 'bzip2' 'attr' 'libldap')
makedepends=('fcgi' 'libmysqlclient' 'lua' 'libxml2' 'e2fsprogs' 'sqlite3' 'gdbm' 'pkgconfig' 'subversion')
backup=('etc/lighttpd/lighttpd.conf' 'etc/logrotate.d/lighttpd')
options=('!libtool' 'emptydirs')
source=(lighttpd.rc.d lighttpd.logrotate.d)
md5sums=('bd690eee0d9e51857448770a151023b0'
         '857e174643fd7761a2f0d8431a679f6c')

_svntrunk="svn://svn.lighttpd.net/lighttpd/trunk/"
_svnmod="lighttpd"

build() {
  cd "$srcdir"

  if [ -d $_svnmod/.svn ]; then
    (cd $_svnmod && svn up -r $pkgver)
  else
    svn co $_svntrunk --config-dir ./ -r $pkgver $_svnmod
  fi

  msg "SVN checkout done or server timeout"
  msg "Starting make..."

  [ -d "$srcdir"/$_svnmod-build ] && rm -r "$srcdir"/$_svnmod-build
  cp -R $_svnmod $_svnmod-build
  cd $_svnmod-build

  # autogen 1.12 drops support for de-ANSI-fication
  sed -i 's/^AM_C_PROTOTYPES/#AM_C_PROTOTYPES/' configure.ac

  ./autogen.sh
  ./configure --prefix=/usr \
    --libexecdir=/usr/lib/lighttpd/modules \
    --sysconfdir=/etc/lighttpd \
    --sharedstatedir=/usr/var \
    --localstatedir=/var \
    --libdir=/usr/lib/lighttpd \
    --includedir=/usr/include/lighttpd \
    --with-mysql \
    --with-ldap \
    --with-attr \
    --with-openssl \
    --with-kerberos5 \
    --without-fam \
    --with-webdav-props \
    --with-webdav-locks \
    --with-gdbm \
    --with-memcache \
    --with-lua

  make
}

package() {
  cd "$srcdir/$_svnmod-build"

  make DESTDIR="$pkgdir" install

  install -D -m755 ../lighttpd.rc.d "$pkgdir"/etc/rc.d/lighttpd
  install -D -m644 ../lighttpd.logrotate.d "$pkgdir"/etc/logrotate.d/lighttpd
  install -d -m755 -o http -g http "$pkgdir"/var/run/lighttpd/
  install -d -m755 -o http -g http "$pkgdir"/var/log/lighttpd/

  install -D -m644 doc/lighttpd.conf "$pkgdir"/etc/lighttpd/lighttpd.conf

  # set sane defaults
  sed -e 's|/srv/www/htdocs/|/srv/http/|' \
      -e 's|/srv/www/|/srv/http/|' \
      -e 's|#server.username            = "wwwrun"|server.username            = "http"|' \
      -e 's|#server.groupname           = "wwwrun"|server.groupname           = "http"|' \
      -e 's|#server.pid-file            = "/var/run/lighttpd.pid"|server.pid-file            = "/var/run/lighttpd/lighttpd.pid"|' \
      -e 's|/usr/local/bin/php-cgi|/usr/bin/php-cgi|' \
      -i "$pkgdir"/etc/lighttpd/lighttpd.conf || return 1

  install -D -m644 COPYING "$pkgdir"/usr/share/licenses/$pkgname/COPYING

  rm -rf "$srcdir"/$_svnmod-build
}

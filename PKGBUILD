# $Id: PKGBUILD 115648 2011-03-18 18:01:46Z heftig $
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Maintainer: tobias [ tobias at archlinux org ]
# Maintainer: Daniel J Griffiths <ghost1227@archlinux.us>

# TMH Modifications:
# _patchlevel: 162, etc. (and corresponding updates to _hgrev)
# breakindent patch from https://retracile.net/attachment/wiki/2010/11/22/23.00/vim-7.3-breakindent.patch?format=raw, updated to build with latest Vim version
# configure --with-compiledby='Taylor M. Hedberg'

pkgbase=vim-tmh-hg
pkgname=(vim-tmh-hg gvim-tmh-hg vim-runtime-tmh-hg)
_topver=7.3
_patchlevel=251
pkgver=4014
pkgrel=3
arch=('i686' 'x86_64')
license=('custom:vim')
url="http://www.vim.org"
makedepends=('gpm' 'perl' 'python2>=2.7.1' 'ruby' 'libxt' 'desktop-file-utils' 'gtk2'
             'gettext' 'pkgconfig' 'mercurial' 'rsync' 'sed')
source=(pythoncomplete.vim::http://www.vim.org/scripts/download_script.php\?src_id=10872
        vimrc archlinux.vim gvim.desktop vim-7.3-breakindent-tmh.patch)

md5sums=('6e7adfbd5d26c1d161030ec203a7f243'
         'e57777374891063b9ca48a1fe392ac05'
         '10353a61aadc3f276692d0e17db1478e'
         '4b83e5fe0e534c53daaba91dd1cd4cbb'
         'f64d88f11a287d0ded23b89df4e1697f')

_hgroot='http://vim.googlecode.com/hg/'
_hgrepo='vim'
__hgbranch='default'

_versiondir="vim${_topver//./}"

##### Build #####

build() {
  cd ${srcdir}

  msg2 'Checking out source from Mercurial...'

  if [[ -d ${_hgrepo} ]]; then
    cd ${_hgrepo}
    hg pull -u || warning 'hg pull failed!'
  else
    hg clone -b ${__hgbranch} "${_hgroot}${_hgrepo}" ${_hgrepo}
    cd ${_hgrepo}
  fi

  _hgid=`hg id -n`
  if (( ${_hgid%+} < $(hg id -nr ${__hgbranch}) )); then
    warning 'You are not building the latest revision!'
    warning "Consider updating _hgrev to $(hg id -r ${__hgbranch})."
    sleep 10
  fi

  cd ..
  rm -rf vim-build gvim-build
  rsync -a --exclude='.hg/' ${_hgrepo}/ vim-build

  msg2 'Patching...'

  # define the place for the global (g)vimrc file (set to /etc/vimrc)
  sed -i 's|^.*\(#define SYS_.*VIMRC_FILE.*"\) .*$|\1|' \
    vim-build/src/feature.h
  sed -i 's|^.*\(#define VIMRC_FILE.*"\) .*$|\1|' \
    vim-build/src/feature.h

  msg2 'Applying breakindent patch...'
  patch -d vim-build -p1 <$srcdir/vim-7.3-breakindent-tmh.patch

  msg2 'Building...'

  cp -a vim-build gvim-build

  cd ${srcdir}/vim-build

  ./configure --prefix=/usr --localstatedir=/var/lib/vim \
    --mandir=/usr/share/man --with-compiledby='Taylor M. Hedberg' \
    --with-features=big --enable-gpm --enable-acl --with-x=no \
    --disable-gui --enable-multibyte --enable-cscope \
    --disable-netbeans --enable-perlinterp --enable-pythoninterp \
    --enable-rubyinterp

  make

  cd ${srcdir}/gvim-build

  ./configure --prefix=/usr --localstatedir=/var/lib/vim \
    --mandir=/usr/share/man --with-compiledby='Taylor M. Hedberg' \
    --with-features=big --enable-gpm --enable-acl --with-x=yes \
    --enable-gui=gtk2 --enable-multibyte --enable-cscope \
    --enable-netbeans --enable-perlinterp --enable-pythoninterp \
    --enable-rubyinterp

  make
}

##### Packaging #####

package_vim-tmh-hg() {
  pkgdesc='Vi Improved, a highly configurable, improved version of the vi text editor'
  depends=("vim-runtime=${pkgver}-${pkgrel}" 'gpm' 'perl' 'python2>=2.7.1' 'ruby')
  conflicts=('gvim' 'gvim-tmh-hg' 'vim')
  provides=("vim=${pkgver}-${pkgrel}")

  cd ${srcdir}/vim-build
  make -j1 VIMRCLOC=/etc DESTDIR=${pkgdir} install

  # provided by (n)vi in core
  rm ${pkgdir}/usr/bin/{ex,view}

  # delete some manpages
  find ${pkgdir}/usr/share/man -type d -name 'man1' 2>/dev/null | \
    while read _mandir; do
    cd ${_mandir}
    rm -f ex.1 view.1 # provided by (n)vi
    rm -f evim.1    # this does not make sense if we have no GUI
  done

  # Runtime provided by runtime package
  rm -r ${pkgdir}/usr/share/vim

  # license
  install -dm755 ${pkgdir}/usr/share/licenses/vim
  ln -s /usr/share/vim/${_versiondir}/doc/uganda.txt \
    ${pkgdir}/usr/share/licenses/vim/license.txt
}

package_gvim-tmh-hg() {
  pkgdesc='Vi Improved, a highly configurable, improved version of the vi text editor (with advanced features, such as a GUI)'
  depends=("vim-runtime=${pkgver}-${pkgrel}" 'gpm' 'perl' 'python2>=2.7.1' 'ruby' 'libxt'
           'desktop-file-utils' 'gtk2')
  provides=("vim=${pkgver}-${pkgrel}" "gvim=${pkgver}-${pkgrel}")
  conflicts=('vim' 'vim-tmh-hg' 'gvim')
  install=gvim.install

  cd ${srcdir}/gvim-build
  make -j1 VIMRCLOC=/etc DESTDIR=${pkgdir} install

  # provided by (n)vi in core
  rm ${pkgdir}/usr/bin/{ex,view}

  # delete some manpages
  find ${pkgdir}/usr/share/man -type d -name 'man1' 2>/dev/null | \
    while read _mandir; do
    cd ${_mandir}
    rm -f ex.1 view.1 # provided by (n)vi
  done

  # Move the runtime for later packaging
  mv ${pkgdir}/usr/share/vim ${srcdir}/runtime-install

  # freedesktop links
  install -Dm644 ${srcdir}/gvim.desktop \
    ${pkgdir}/usr/share/applications/gvim.desktop
  install -Dm644 runtime/vim48x48.png ${pkgdir}/usr/share/pixmaps/gvim.png

  # license
  install -dm755 ${pkgdir}/usr/share/licenses/gvim
  ln -s /usr/share/vim/${_versiondir}/doc/uganda.txt \
    ${pkgdir}/usr/share/licenses/gvim/license.txt
}

package_vim-runtime-tmh-hg() {
  pkgdesc='Runtime for vim and gvim'
  backup=(etc/vimrc)
  provides=("vim-runtime=${pkgver}-${pkgrel}")
  conflicts=('vim-runtime')

  # Install the runtime split from gvim
  install -dm755 ${pkgdir}/usr/share
  mv ${srcdir}/runtime-install ${pkgdir}/usr/share/vim

  # Don't forget logtalk.dict
  install -Dm644 ${srcdir}/gvim-build/runtime/ftplugin/logtalk.dict \
    ${pkgdir}/usr/share/vim/${_versiondir}/ftplugin/logtalk.dict

  # fix FS#17216
  sed -i 's|messages,/var|messages,/var/log/messages.log,/var|' \
    ${pkgdir}/usr/share/vim/${_versiondir}/filetype.vim

  # patch filetype.vim for better handling of pacman related files
  sed -i "s/rpmsave/pacsave/;s/rpmnew/pacnew/;s/,\*\.ebuild/\0,PKGBUILD*,*.install/" \
    ${pkgdir}/usr/share/vim/${_versiondir}/filetype.vim
  sed -i "/find the end/,+3{s/changelog_date_entry_search/changelog_date_end_entry_search/}" \
    ${pkgdir}/usr/share/vim/${_versiondir}/ftplugin/changelog.vim

  # make Aaron happy
  install -Dm644 ${srcdir}/pythoncomplete.vim \
    ${pkgdir}/usr/share/vim/${_versiondir}/autoload/pythoncomplete.vim
  
  # rc files
  install -Dm644 ${srcdir}/vimrc ${pkgdir}/etc/vimrc
  install -Dm644 ${srcdir}/archlinux.vim \
    ${pkgdir}/usr/share/vim/vimfiles/archlinux.vim

  # license
  install -dm755 ${pkgdir}/usr/share/licenses/vim-runtime
  ln -s /usr/share/vim/${_versiondir}/doc/uganda.txt \
    ${pkgdir}/usr/share/licenses/vim-runtime/license.txt
}

# vim:set sw=2 sts=2 et:

# $Id: bfc54885f6617a81b35939f12f0a9f02199bb6d8 $
# Maintainer: Ido Rosen <ido@kernel.org>
# Co-maintainer: Chris Fordham <chris at fordham-nagy dot id dot au> aka flaccid
# Contributor: Sébastien "Seblu" Luttringer
# Contributor: Marcel Wysocki <maci@satgnu.net>
# Contributor: Daniel YC Lin <dlin.tw@gmail>
# Contributor: Joerg <joerg@higgsboson.tk>
# Contributor: Vincent Aranega <vincent.aranega@gmail.com>
#
# NOTE: To request changes to this package, please submit a pull request
#       to the GitHub repository at https://github.com/ido/packages-archlinux
#       Otherwise, open a GitHub issue.  Thank you! -Ido
#

# Important upstream docs:
# https://github.com/docker/docker/blob/master/project/PACKAGERS.md

pkgname=docker-git
pkgver=1.11.0
pkgrel=1
epoch=1
pkgdesc='Pack, ship and run any application as a lightweight container.'
arch=('i686' 'x86_64')
url="https://github.com/docker/docker"
license=('Apache License Version 2.0')
depends=('bridge-utils' 'iproute2' 'device-mapper' 'sqlite' 'systemd')
makedepends=('git' 'go' 'btrfs-progs' 'go-md2man' 'apparmor-libapparmor')
backup=(etc/sysctl.d/docker.conf)
provides=('docker')
conflicts=('docker')
# don't strip binaries! A sha1 is used to check binary consistency.
options=('!strip')
source=("git+https://github.com/docker/docker.git"
        "git+https://github.com/docker/containerd.git"
        "git+https://github.com/opencontainers/runc.git"
        'docker.service'
        'docker.install'
        docker.conf
        )
md5sums=('SKIP'
         'SKIP'
         'SKIP'
         'f95610c9cb86cb617371040dad5b95d3'
         '1a8e60447794b3c4f87a2272cc9f144f'
         '9bce988683771fb8262197f2d8196202')
install='docker.install'

prepare() {
  cd docker
}

build() {
  export GOPATH=${PWD}/go
  export OLD_LDFLAGS=${LDFLAGS}
  export LDFLAGS=""

  # runc
  export RUNC_COMMIT=e87436998478d222be209707503c27f6f91be0c5
  mkdir -p $GOPATH/src/github.com/opencontainers/
  ln -s ${PWD}/runc $GOPATH/src/github.com/opencontainers/runc
  pushd "$GOPATH/src/github.com/opencontainers/runc"
  git checkout -q "$RUNC_COMMIT" 
  make BUILDTAGS="seccomp apparmor selinux" 
  popd

  # containerd
  export CONTAINERD_COMMIT=d2f03861c91edaafdcb3961461bf82ae83785ed7
  mkdir -p $GOPATH/src/github.com/docker/
  ln -s ${PWD}/containerd $GOPATH/src/github.com/docker/containerd 
  pushd "$GOPATH/src/github.com/docker/containerd"
  pwd
  git checkout -q "$CONTAINERD_COMMIT"
  make static
 
  export LDFLAGS=${OLD_LDFLAGS}
  popd
  #cd "$_magic/docker"
  #export GOPATH="$srcdir:$srcdir/$_magic/docker/vendor"
  cd docker
  git checkout -q v1.11.0
  export AUTO_GOPATH=1
  ./hack/make.sh dynbinary
  for i in man/*.md; do
    go-md2man -in "$i" -out "${i%.md}"
  done
}

#check() {
  #cd "$_magic/docker"
  # Will be added upstream soon
  #./hack/make.sh dyntest
#}

package() {
  #cd "$_magic/docker"
  install -Dm755 "runc/runc" "$pkgdir/usr/local/bin/docker-runc"

  install -Dm755 "containerd/bin/containerd" "$pkgdir/usr/local/bin/docker-containerd"
  install -Dm755 "containerd/bin/containerd-shim" "$pkgdir/usr/local/bin/docker-containerd-shim"
  install -Dm755 "containerd/bin/ctr" "$pkgdir/usr/local/bin/docker-containerd-ctr"

  cd docker
  _dockerver="$(cat VERSION)"
  install -Dm755 "bundles/$_dockerver/dynbinary/docker-$_dockerver" "$pkgdir/usr/bin/docker"

  # completion
  install -Dm644 "contrib/completion/bash/docker" "$pkgdir/usr/share/bash-completion/completions/docker"
  install -Dm644 "contrib/completion/zsh/_docker" "$pkgdir/usr/share/zsh/site-functions/_docker"

  # systemd
  install -Dm644 "$srcdir/docker.service" "$pkgdir/usr/lib/systemd/system/docker.service"
  install -Dm644 "$srcdir/docker.conf" "$pkgdir/etc/sysctl.d/docker.conf"

  cd man
  for section in 1 5; do
    for i in *.$section; do
      install -Dm644 "$i" "$pkgdir/usr/share/man/man$section/$i"
    done
  done
}

# vim:set ts=2 sw=2 et:

# Maintainer: Simao Reis <smnrsti@gmail.com>
pkgname=forked-tls-go
pkgver=1.5.3
pkgrel=0
pkgdesc="Go programming language compiler"
url="http://www.golang.org/"
arch="x86 x86_64 armhf"
license="BSD"
depends=""
depends_dev=""
makedepends="bash perl mercurial go-bootstrap"
options="!strip"
install=""
subpackages="
	$pkgname-tools:tools
	"
source="
	https://github.com/sreis/go/archive/go1.5-SNAPSHOT.tar.gz
	"

_builddir="$srcdir"/go-go1.5-SNAPSHOT
_toolrepo=golang.org/x/tools
_tooltag=release-branch.go1.4

prepare() {
	local i
	cd "$_builddir"
	for i in $source; do
		case $i in
		*.patch) msg $i; patch -p1 -i "$srcdir"/$i || return 1;;
		esac
	done
}

build() {
	export GOPATH="$srcdir"
	export GOROOT="$_builddir"
	export GOROOT_BOOTSTRAP="/usr/lib/go-bootstrap"
	export GOBIN="$GOROOT"/bin
	export GOROOT_FINAL=/usr/lib/go
	# ccache breaks build for some reason
	unset CC

	case "$CARCH" in
	x86) GOARCH=386;;
	x86_64) GOARCH=amd64;;
	arm*) GOARCH=arm; export GOARM=6;;
	*) return 1;;
	esac
	export GOARCH

	cd "$_builddir/src"

	./make.bash || return 1

	# FIXME: race and bench tests fail:
	#PATH="$GOROOT/bin:$PATH" ./run.bash --no-rebuild --banner || return 1

	# Build tools provided with the upstream binary distribution:
	"$GOROOT"/bin/go get -d $_toolrepo/...
	(
		cd "$srcdir"/src/$_toolrepo
		git checkout $_tooltag
	)
	local tool
	for tool in cover vet godoc; do
		"$GOROOT"/bin/go install $_toolrepo/cmd/$tool
	done
}

package() {
	local f d
	cd "$_builddir"

	install -dm755 "$pkgdir"/usr/bin
	for f in go gofmt; do
		install -m755 bin/$f "$pkgdir"/usr/bin
	done

	install -dm755 "$pkgdir"/usr/lib/go
	cp -a pkg "$pkgdir"/usr/lib/go

	# The source needs to be installed due to an upstream
	# bug (http://code.google.com/p/go/issues/detail?id=2775).
	# When this is resolved we can split out the source to a
	# go-doc sub package.
	install -dm755 "$pkgdir"/usr/lib/go/src
	for d in pkg cmd; do
		cp -a src/$d "$pkgdir"/usr/lib/go/src
	done
}

tools() {
	local tool

	pkgdesc="Go programming language tools"
	depends="$pkgname"

	install -Dm755 "$_builddir"/bin/godoc "$subpkgdir"/usr/bin/godoc

	install -dm755 "$subpkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH

	for tool in cover vet; do
		mv "$pkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH/$tool \
			"$subpkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH/$tool
	done

	# Make documentation for tools available through godoc:
	for tool in cover vet godoc; do
		install -dm755 "$subpkgdir"/usr/lib/go/src/pkg/$tool
		sed -e 's/^package main$/package documentation/' \
			"$srcdir"/src/$_toolrepo/cmd/$tool/doc.go > \
			"$subpkgdir"/usr/lib/go/src/pkg/$tool/doc.go
	done
}

md5sums="ef8aa8a7c357fdd9dde4b0ba14bb93ba  go1.5-SNAPSHOT.tar.gz"
sha256sums="b281f54575ef34039ea955ff502e9f7ac84bc8e02e02d6bc9267a01a067d5f81  go1.5-SNAPSHOT.tar.gz"
sha512sums="2228f13eceb71bb9bcffa46ea23e0db20499e83bb170890c1d1e68e18d53a31e5619919d2928bebc8eb0dcfae2f214978be1caa0f97d059504b17ea2301ce5ef  go1.5-SNAPSHOT.tar.gz"

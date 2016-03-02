# Maintainer: Simao Reis <smnrsti@gmail.com>
pkgname=forked-tls-go
pkgver=1.6.0
pkgrel=0
pkgdesc="Forked version of Go programming language compiler with TLS patch."
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
	https://github.com/sreis/go/archive/go1.6-SNAPSHOT.tar.gz
	"

_builddir="$srcdir"/go-go1.6-SNAPSHOT
_toolrepo=golang.org/x/tools
_tooltag=release-branch.go1.4

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
	for d in $(find src/ -type d -print); do
		cp -a $d "$pkgdir"/usr/lib/go/src
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

md5sums="10d6902f4392baa85dd4889044bdd37c  go1.6-SNAPSHOT.tar.gz"
sha256sums="32ef677ba2e429a5c6fc5795ff650454429a9bcc8f2866f1637d8b9f441b97d1  go1.6-SNAPSHOT.tar.gz"
sha512sums="6a2b62f82636df56d37bb8756ec09ee3cbc047a1ad5d32241422cf6d00777d227258c29b3f013f679f5872a219cad9458ee2438bba31bf514b632c157a67147b  go1.6-SNAPSHOT.tar.gz"

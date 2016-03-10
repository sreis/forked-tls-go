# Maintainer: Simão Reis <smnrsti@gmail.com>
pkgname=go-forked-tls
pkgver=1.6
# This should be the latest commit on the corresponding release branch
_toolsver="c887be1b2ebd11663d4bf2fbca508c449172339e"
pkgrel=1
pkgdesc="Go programming language compiler with TLS patch."
url="http://www.golang.org/"
arch="x86 x86_64 armhf"
license="BSD"
depends=""
depends_dev=""
makedepends="bash go-bootstrap"
options="!strip"
install=""
subpackages="go-tools"
source="https://github.com/sreis/go/archive/go1.6-SNAPSHOT.tar.gz
	go-tools-$pkgver.tar.gz::https://github.com/golang/tools/archive/${_toolsver}.tar.gz
	no-pic.patch
	fix-musl.patch"

# NOTE: building go for x86 with grsec kernel requires:
#           sysctl -w kernel.modify_ldt=1

_gotools="cover vet godoc"

_builddir="$srcdir"/go-go1.6-SNAPSHOT
_tooldir="$srcdir"/tools-${_toolsver}

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
	cd "$_builddir/src"

	export GOPATH="$srcdir"
	export GOROOT="$_builddir"
	export GOBIN="$GOROOT"/bin
	export GOROOT_FINAL=/usr/lib/go
	export GOROOT_BOOTSTRAP=/usr/lib/go-bootstrap

	case "$CARCH" in
	x86)	export GOARCH="386" ;;
	x86_64)	export GOARCH="amd64" ;;
	arm*)	export GOARCH="arm" ;;
	*)	return 1 ;;
	esac

	./make.bash --no-clean || return 1

	# FIXME some tests fail:
	# PATH="$GOROOT/bin:$PATH" ./run.bash -no-rebuild || return 1

	mkdir -p "$GOPATH"/src/golang.org/x/tools
	cp -r "$_tooldir"/* "$GOPATH"/src/golang.org/x/tools

	for tool in $_gotools; do
		"$GOROOT"/bin/go install \
			golang.org/x/tools/cmd/$tool || return 1
	done
}

package() {
	cd "$_builddir"
	mkdir -p "$pkgdir"/usr/bin "$pkgdir"/usr/lib/go/bin "$pkgdir"/usr/share/doc/go

	for binary in go gofmt; do
		mv bin/"$binary" "$pkgdir"/usr/lib/go/bin/ || return 1
		ln -s /usr/lib/go/bin/"$binary" "$pkgdir"/usr/bin/ || return 1
	done

	# The source needs to be installed due to an upstream
	# bug (https://github.com/golang/go/issues/2775).
	# When this is resolved we can split out the source to a
	# go-doc sub package.
	cp -a pkg src "$pkgdir"/usr/lib/go || return 1
	cp -r doc misc "$pkgdir"/usr/share/doc/go || return 1

	# Remove tests from /usr/lib/go/src.
	# Those shouldn't be affacted by the upstream bug (see above).
	find "$pkgdir"/usr/lib/go/src \( -type f -a -name "*_test.go" \) \
		-exec rm -rf \{\} \+ || return 1
	find "$pkgdir"/usr/lib/go/src \( -type d -a -name "testdata" \) \
		-exec rm -rf \{\} \+ || return 1
	find "$pkgdir"/usr/lib/go/src \( -type f -a -name "*.bash" \) \
		-exec rm -rf \{\} \+ || return 1

	rm -rf "$pkgdir"/usr/lib/go/pkg/bootstrap
	rm -f  "$pkgdir"/usr/lib/go/pkg/tool/*/api
}

tools() {
	pkgdesc="Go programming language tools"
	depends="$pkgname"

	mkdir -p "$subpkgdir"/usr/bin "$subpkgdir"/usr/lib/go/bin
	for binary in "$_builddir"/bin/*; do
		mv $binary "$subpkgdir"/usr/lib/go/bin || return 1
		ln -s /usr/lib/go/bin/"${binary##*/}" "$subpkgdir"/usr/bin/ || return 1
	done

	mkdir -p "$subpkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH
	cp "$subpkgdir"/usr/lib/go/bin/godoc \
		"$pkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH/godoc || return 1

	for tool in $_gotools; do
		mv "$pkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH/$tool \
			"$subpkgdir"/usr/lib/go/pkg/tool/linux_$GOARCH/$tool || return 1
	done
}

md5sums="10d6902f4392baa85dd4889044bdd37c  go1.6-SNAPSHOT.tar.gz
733a96b59562ed84ff552542f16a3ab3  go-tools-1.6.tar.gz
f54c834b108eb8a82f837790dbb9c6cd  no-pic.patch
f1870044d54331f8890314f0f0c0c7f7  fix-musl.patch"
sha256sums="32ef677ba2e429a5c6fc5795ff650454429a9bcc8f2866f1637d8b9f441b97d1  go1.6-SNAPSHOT.tar.gz
955e5a119babad356d9cf00cdaaf3c27648d4451109b467c872fad8cc94d5b56  go-tools-1.6.tar.gz
3cec4410cf1d14bb0895307e2ab88b62890f3b8937063a4539bd498c039a5fd8  no-pic.patch
e9ce32af05d71d43f57034576a06bf503eb7848076d3457121822f580bcd9f17  fix-musl.patch"
sha512sums="6a2b62f82636df56d37bb8756ec09ee3cbc047a1ad5d32241422cf6d00777d227258c29b3f013f679f5872a219cad9458ee2438bba31bf514b632c157a67147b  go1.6-SNAPSHOT.tar.gz
955def7cc3e5ceb5d8e47477400007d32c25f3bc2764f04a0451bd0235af507e63a0966d201a345f51ac764471da8620b5357d1099ec39e2422bd04d25fd71b8  go-tools-1.6.tar.gz
c042e1efd13ba17870f71d979798b6adbfa6fddef11792b0c18187116593887aa4b3c5454a315c452a386bdd9a0643df3a70c684490ede5e51363e143812b578  no-pic.patch
7cce40e8c43d9b0e8b459d3f33f38c96a08faf1d05b83457db52a26596e2beac637defd9395f84ea436edfc1f0d49a2e63d102b4587d8a035d0466f716535eed  fix-musl.patch"

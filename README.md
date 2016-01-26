# forked-tls-go
Alpine package build instructions for a patched version of `go`.

### Instructions

More info here: 
- http://wiki.alpinelinux.org/wiki/Installation
- http://wiki.alpinelinux.org/wiki/Install_Alpine_on_VirtualBox
- http://wiki.alpinelinux.org/wiki/Creating_an_Alpine_package

#### Install Alpine in a VirtualBox VM
#### Create user and configure sudo
#### Install dependencies

```bash
apk add bash mercurial perl go-bootstrap 
```

#### Build instructions

```
git clone github.com/sreis/forked-tls-go
cd forked-tls-go
abuild -r
```

This will create a `~/packages` with the apk package.

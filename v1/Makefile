# Makefile for placemat

SUDO = sudo
FAKEROOT = fakeroot

### for debian package
PACKAGES := fakeroot
WORKDIR := $(CURDIR)/work
CONTROL := $(WORKDIR)/DEBIAN/control
DOCDIR := $(WORKDIR)/usr/share/doc/placemat
EXAMPLEDIR := $(WORKDIR)/usr/share/doc/placemat/examples
BASH_COMPLETION_DIR := $(WORKDIR)/etc/bash_completion.d
BINDIR := $(WORKDIR)/usr/bin
VERSION = v0.1
DEB_VERSION = $(patsubst v%,%,$(VERSION))
DEB = placemat_$(DEB_VERSION)_amd64.deb
DEST = .
SBIN_PKGS = ./pkg/placemat ./pkg/pmctl

all: test

test:
	test -z "$$(gofmt -s -l . | tee /dev/stderr)"
	test -z "$$(golint $$(go list ./...) | grep -v '/mtest/.*: should not use dot imports' | tee /dev/stderr)"
	test -z "$$(nilerr ./... 2>&1 | tee /dev/stderr)"
	test -z "$$(custom-checker -restrictpkg.packages=html/template,log $$(go list -tags='$(GOTAGS)' ./...) 2>&1 | tee /dev/stderr)"
	ineffassign .
	go build ./...
	go test -race -v ./...
	go vet ./...

mod:
	go mod tidy
	git add go.mod

deb: $(DEB)

$(DEB):
	rm -rf $(WORKDIR)
	cp -r debian $(WORKDIR)
	sed 's/@VERSION@/$(DEB_VERSION)/' debian/DEBIAN/control > $(CONTROL)
	mkdir -p $(BINDIR)
	GOBIN=$(BINDIR) go install $(SBIN_PKGS)
	mkdir -p $(DOCDIR)
	cp ../README.md ../LICENSE ../docs/pmctl.md $(DOCDIR)
	cp -r ../examples $(DOCDIR)
	mkdir -p $(BASH_COMPLETION_DIR)
	$(BINDIR)/pmctl completion > $(BASH_COMPLETION_DIR)/placemat
	chmod -R g-w $(WORKDIR)
	$(FAKEROOT) dpkg-deb --build $(WORKDIR) $(DEST)
	rm -rf $(WORKDIR)

setup:
	GO111MODULE=off go get -u golang.org/x/lint/golint
	$(SUDO) apt-get update
	$(SUDO) apt-get -y install --no-install-recommends $(PACKAGES)

clean:
	rm -rf $(WORKDIR) $(DEB)

.PHONY:	all test mod deb setup clean

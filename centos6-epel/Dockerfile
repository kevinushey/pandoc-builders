FROM centos:centos6

## Add epel repositories

RUN yum install -y epel-release

## This is a custom version of stack, which links to
## and older version of libgmp, the one that is available
## on CentOS 6

RUN curl -sSL https://s3.amazonaws.com/download.fpcomplete.com/centos/6/fpco.repo | \
        tee /etc/yum.repos.d/fpco.repo
RUN yum -y install stack

## Download pandoc-citeproc source code from a release.
## This will build both pandoc-citeproc and pandoc

RUN curl -LO https://hackage.haskell.org/package/pandoc-citeproc-0.10.4/pandoc-citeproc-0.10.4.tar.gz

## Setup stack. This will download and install a GHC compiler
## with the full set of tools needed.
## Need to run a newer plan, to avoid a tls bug that was fixed
## in version 1.3.10.

RUN tar xzf pandoc-citeproc-0.10.4.tar.gz && \
    cd pandoc-citeproc-0.10.4 && \
    cat stack.yaml | sed 's/^- pandoc-1.19.1/- pandoc-1.19.2.1/' | \
    sed 's/^resolver: lts-7.14/resolver: lts-8.12/' > stack.yaml2 && \
    mv stack.yaml2 stack.yaml && \
    stack setup

## Extra packages needed for the actual build

RUN yum -y install git zlib-devel

## Build the binaries. The result will be in /root/.local/bin,
## two files: pandoc and pandoc-citeproc
## These are shared binaries, but otherwise they are self-contained.

RUN cd pandoc-citeproc-0.10.4 && \
    stack install --flag pandoc-citeproc:embed_data_files

## The same for pandoc

RUN curl -LO https://hackage.haskell.org/package/pandoc-1.19.2.1/pandoc-1.19.2.1.tar.gz && \
    tar xzf pandoc-1.19.2.1.tar.gz && \
    cd pandoc-1.19.2.1 && \
    cat stack.yaml | sed 's/^resolver: lts-[0-9][0-9]*\.[0-9][0-9]*/resolver: lts-8.12/' > stack.yaml2 && \
    mv stack.yaml2 stack.yaml && \
    stack setup && \
    stack install --flag pandoc:embed_data_files

## Now we build an RPM package with the two binary files

RUN yum -y install rpm-build &&                             \
    cd /root &&                                             \
    mkdir -p rpmbuild/{BUILD,RPMS,SPECS,SRPMS,SOURCES} &&   \
    cd .local &&                                            \
    tar czf /root/rpmbuild/SOURCES/source.tar.gz bin
    
ADD pandoc.spec /root/rpmbuild/SPECS/pandoc.spec
ADD pandoc-citeproc.spec /root/rpmbuild/SPECS/pandoc-citeproc.spec

RUN cd /root && rpmbuild --define "_topdir `pwd`/rpmbuild" -bb rpmbuild/SPECS/pandoc.spec
RUN cd /root && rpmbuild --define "_topdir `pwd`/rpmbuild" -bb rpmbuild/SPECS/pandoc-citeproc.spec

## Create a repository

RUN yum -y install createrepo &&                \
    cd /root &&                                 \
    mkdir centos6 &&                            \
    cp rpmbuild/RPMS/x86_64/pandoc-* centos6 && \
    createrepo centos6

ADD rhub-pandoc-centos6.repo /root/centos6/rhub-pandoc-centos6.repo

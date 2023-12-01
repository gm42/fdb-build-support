###
### This Dockerfile is based upon the centos7 version, adjusted to work with Ubuntu 22 as base image
### and is useful only to generate a minimal version of FoundationDB (no Java or C# bindings).
###

FROM ubuntu:22.04

LABEL org.opencontainers.image.authors="16498973+gm42@users.noreply.github.com"

RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update && apt-get --no-install-recommends install -y \
    bzip2 ca-certificates adduser apt base-files base-passwd bash binutils build-essential cpp \
    dpkg dos2unix elfutils fakeroot findutils g++ gawk gdb git \
    grep gzip hostname libtsan0 sed tar \
    clang-14 lld-14 \
    locales login m4 make makedev mawk passwd \
    texinfo udev unzip util-linux valgrind wget curl sphinx-common gnupg python3 \
    cmake ninja-build liblz4-dev libjemalloc-dev && \
    rm -rf /var/lib/apt/lists/* && \
    LANGUAGE=en_US.UTF-8 LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 locale-gen en_US.UTF-8 && \
    dpkg-reconfigure locales

RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF && \
    echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" > /etc/apt/sources.list.d/mono-official-stable.list && \
    apt update && \
    apt-get install mono-devel -y && \
    rm -rf /var/lib/apt/lists/*

RUN curl -Ls https://boostorg.jfrog.io/artifactory/main/release/1.78.0/source/boost_1_78_0.tar.bz2 -o boost_1_78_0.tar.bz2 && \
    echo "8681f175d4bdb26c52222665793eef08490d7758529330f98d3b29dd0735bccc  boost_1_78_0.tar.bz2" > boost-sha.txt && \
    sha256sum --quiet -c boost-sha.txt && \
    mkdir -p /opt/boost_1_78_0 && \
    tar --strip-components 1 --no-same-owner --directory /opt/boost_1_78_0 -xjf boost_1_78_0.tar.bz2 && \
    rm boost_1_78_0.tar.bz2 && \
    cp -R /opt/boost_1_78_0 /opt/boost_1_78_0_clang

# install Boost to /opt
# This used to pass --with-libraries=context,filesystem,iostreams,system,serialization
# which doesn't build serialization (or builds it in a form that downstream cmake builds
# cannot find).  Instead of debugging their build logic, we pass "all", which is simpler to
# maintain and does build serialization.
RUN cd /opt/boost_1_78_0 && \
    ./bootstrap.sh --with-libraries=all && \
    ./b2 link=static cxxflags="-std=c++14 -fPIC" --prefix=/opt/boost_1_78_0 -j32 install && \
    rm -rf /opt/boost_1_78_0/libs && \
    rm -rf /tmp/*

# install Boost to /opt, using clang to compile the library
# Boost::context depends on some C++11 features, e.g. std::call_once; however,
# gcc and clang are using different ABIs, thus a gcc-built Boost::context is
# not linkable to clang objects.
RUN export PATH="$PATH:/usr/lib/llvm-14/bin" && \
    cd /opt/boost_1_78_0_clang && \
    ./bootstrap.sh --with-toolset=clang --with-libraries=all && \
    ./b2 link=static cxxflags="-std=c++14 -I/usr/include/c++/11 -I/usr/include/aarch64-linux-gnu/c++/11 -stdlib=libc++ -nostdlib++ -fPIC" linkflags="-stdlib=libc++ -nostdlib++ -static-libgcc -lc++ -lc++abi" --prefix=/opt/boost_1_78_0_clang -j32 install && \
    rm -rf /opt/boost_1_78_0_clang/libs && \
    rm -rf /tmp/*

RUN adduser --disabled-password --gecos '' fdb && chown -R fdb /opt && chmod -R 0777 /opt

USER fdb

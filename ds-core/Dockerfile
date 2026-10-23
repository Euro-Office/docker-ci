# Toolchain for building Euro-Office core (server flavour), mirroring the
# vcpkg-base, vcpkg-local and core-base stages of core/.docker/core.bake.Dockerfile.
#
# Ubuntu 22.04 keeps the output on glibc 2.35, see that file for the rationale.
# The vcpkg binary cache is a local NuGet source at /nuget-cache, mount a volume
# there to reuse built third-party packages across runs.
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -yq --no-install-recommends \
        ca-certificates git curl wget ssh gpg sudo ccache \
        zip unzip tar \
        build-essential make cmake ninja-build pkg-config \
        mono-devel \
        libglib2.0-dev \
        python3 python-is-python3 python3-venv python3-setuptools python3-httplib2 \
        lsb-release autoconf automake libtool findutils \
        gn \
    && rm -rf /var/lib/apt/lists/*

# clang-13 required for V8 9.x
RUN wget -qO - https://apt.llvm.org/llvm-snapshot.gpg.key | \
        gpg --dearmor -o /etc/apt/keyrings/llvm-snapshot.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/llvm-snapshot.gpg] http://apt.llvm.org/jammy/ llvm-toolchain-jammy-13 main" \
        > /etc/apt/sources.list.d/llvm-13.list && \
    apt-get update && apt-get install -yq --no-install-recommends \
        clang-13 lld-13 llvm-13-dev llvm-13 \
        libc++-13-dev libc++abi-13-dev \
        qemu-user-static binfmt-support && \
    rm -rf /var/lib/apt/lists/* && \
    for tool in clang clang++ llvm-ar llvm-nm llvm-ranlib lld; do \
        update-alternatives --install /usr/bin/$tool $tool /usr/bin/$tool-13 100; \
    done

ENV VCPKG_ROOT=/opt/vcpkg
ENV PATH="${VCPKG_ROOT}:${PATH}"
ENV VCPKG_BINARY_SOURCES="clear;nuget,NuGetCache,readwrite;nugettimeout,3600"

RUN git clone https://github.com/microsoft/vcpkg.git ${VCPKG_ROOT} && \
    ${VCPKG_ROOT}/bootstrap-vcpkg.sh && \
    vcpkg fetch nuget && \
    mkdir /nuget-cache && \
    mono $(vcpkg fetch nuget) sources add -Source /nuget-cache -Name NuGetCache

RUN git config --global --add safe.directory '*'

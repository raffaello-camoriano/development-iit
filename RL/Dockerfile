ARG from=diegoferigo/development:nvidia-master
FROM ${from}

# Build-time metadata as defined at http://label-schema.org
# ARG BUILD_DATE
# ARG VCS_REF
# ARG VERSION=0.9
LABEL \
    maintainer="Diego Ferigo <dgferigo@gmail.com>" \
    # org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.name="rl" \
    org.label-schema.description="" \
    org.label-schema.url="https://github.com/diegoferigo/development-iit" \
    # org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url="https://github.com/diegoferigo/development-iit" \
    # org.label-schema.vendor="" \
    # org.label-schema.version=$VERSION \
    org.label-schema.schema-version="1.0"

ENV RL_DIR=/rl
ENV RL_INSTALL=${RL_DIR}/local
ENV RL_SOURCES=${RL_DIR}/sources
ENV DEVENV_PATH=${RL_INSTALL}/bin:${DEVENV_PATH}
ENV CMAKE_PREFIX_PATH=${CMAKE_PREFIX_PATH}:${RL_INSTALL}
ENV LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${RL_INSTALL}/lib

# =======================================
# Install ignition libraries from sources
# =======================================

# http://gazebosim.org/tutorials?tut=install_dependencies_from_source

ENV CC="gcc-8"
ENV CXX="g++-8"

# Common tools
RUN apt-get update &&\
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        apt-utils \
        gnupg \
        gnupg2 \
        apt-utils \
        ca-certificates \
        locales \
        curl \
        lsb-release \
        software-properties-common \
        wget \
        libxml2-dev \
        python3-tk \
        ssh \
        &&\
    rm -rf /var/lib/apt/lists/*

# Development tools
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | apt-key add - &&\
    apt-add-repository "deb https://apt.kitware.com/ubuntu/ `lsb_release -cs` main" &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        git \
        cmake \
        cmake-curses-gui \
        build-essential \
        gcc-8 \
        g++-8 \
        ninja-build \
        mercurial \
        python3-pip \
        xvfb \
        &&\
    rm -rf /var/lib/apt/lists/*

# Install protobuf
RUN apt-get update &&\
    apt-get install -y --no-install-recommends \
        libtool \
        autoconf \
        automake \
        &&\
    rm -rf /var/lib/apt/lists/* &&\
    cd /tmp &&\
    git clone --depth 1 -b v3.8.0 https://github.com/google/protobuf &&\
    cd protobuf &&\
    git submodule update --init --recursive &&\
    ./autogen.sh &&\
    ./configure &&\
    make -j8 &&\
    make install &&\
    rm -r /tmp/protobuf
    
# Uninstall gazebo classic and protobuf
RUN apt remove -y gazebo10 libgazebo10-dev libprotobuf-dev protobuf-compiler libprotobuf10 libprotobuf-lite10 &&\
    apt autoremove -y

# Ignition dependencies
RUN apt-get update &&\
    apt-get install -y --no-install-recommends \
        ruby-dev \
        ruby \
        # ign-common
        libtinyxml2-dev \
        libswscale-dev \
        libavdevice-dev \
        libavformat-dev \
        libavcodec-dev \
        libavutil-dev \
        libfreeimage-dev \
        libgts-dev \
        libsqlite3-dev \
        # sdformat
        python \
        libboost-system-dev \
        libxml2-utils \
        # ign-fuel-tools
        libjsoncpp-dev \
        libzip-dev \
        libyaml-dev \
        libcurl4-openssl-dev \
        # ign-transport
        libzmqpp-dev \
        uuid-dev \
        # ign-rendering
        libogre-1.9-dev \
        # ign-tools
        libwebsockets-dev \
        #ign-gui
        qtquickcontrols2-5-dev \
        # ign-gazebo compile deps
        libgflags-dev \
        # ign-gazebo runtime deps
        qtbase5-dev \
        qtdeclarative5-dev \
        qml-module-qtquick2 \
        qml-module-qtquick-controls \
        qml-module-qtquick-controls2 \
        qml-module-qtquick-dialogs \
        qml-module-qtquick-layouts \
        qml-module-qt-labs-folderlistmodel \
        qml-module-qt-labs-settings \
        &&\
    rm -rf /var/lib/apt/lists/*

# Setup locales and tzdata
RUN sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen &&\
    locale-gen &&\
    update-locale LANG="en_US.UTF-8" &&\
    export DEBIAN_FRONTEND=noninteractive &&\
    export DEBCONF_NONINTERACTIVE_SEEN=true &&\
    rm -rf /etc/localtime &&\
    rm -rf /etc/timezone &&\
    echo 'tzdata tzdata/Areas select Europe' | debconf-set-selections &&\
    echo 'tzdata tzdata/Zones/Europe select Rome' | debconf-set-selections &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends tzdata &&\
    rm -rf /var/lib/apt/lists/*

# Add gazebo ppa and install other deps.
# We want to install from source ign-gazebo, ign-physics, sdformat, and all the components that depend on protobuf.
RUN echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list &&\
    wget http://packages.osrfoundation.org/gazebo.key -O - | apt-key add - &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        libogre-2.1-dev \
        libdart6-all-dev \
        # Install from ppa base components
        libignition-cmake2-dev \
        libignition-common3-dev \
        libignition-math6-dev \
        libignition-math6-eigen3-dev \
        libignition-plugin-dev \
        libignition-rendering3 \
        libignition-rendering3-dev \
        libignition-tools-dev \
        &&\
    rm -rf /var/lib/apt/lists/*

RUN wget https://raw.github.com/mnauw/git-remote-hg/master/git-remote-hg -O /usr/local/bin/git-remote-hg &&\
    chmod +x /usr/local/bin/git-remote-hg

# Create a new runtimeusers group (it might be already present)
RUN groupadd runtimeusers || true

# Download all the sources
ARG RL_SOURCES="20200121"
RUN mkdir -p ${RL_SOURCES} &&\
    cd ${RL_SOURCES} &&\
    mkdir -p ${RL_SOURCES} &&\
    git config --global remote-hg.ignore-name "~|pre|pendulum" &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-transport -b branches/ign-transport8 && cd ign-transport && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-msgs -b branches/ign-msgs5 && cd ign-msgs && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/osrf/sdformat -b branches/sdf9 && cd sdformat && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-physics -b branches/ign-physics2 && cd ign-physics && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-sensors -b branches/ign-sensors3 && cd ign-sensors && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-gui -b branches/ign-gui3 && cd ign-gui && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-gazebo -b branches/ign-gazebo3 && cd ign-gazebo && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-launch -b branches/ign-launch2 && cd ign-launch && git gc --aggressive && cd - &&\
    git clone hg::https://bitbucket.org/ignitionrobotics/ign-fuel-tools -b branches/ign-fuel-tools4 && cd ign-fuel-tools && git gc --aggressive && cd - &&\
    git clone https://github.com/robotology/icub-gazebo &&\
    chgrp -R runtimeusers ${RL_DIR} && chmod -R g+rw ${RL_DIR}

ENV CMAKE_GENERATOR="Ninja"
ENV SOURCES_BUILD_TYPE="RelWithDebInfo"

# sdformat
RUN cd ${RL_SOURCES}/sdformat &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        # TODO: build tests OFF?
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+
    # TODO: with clang here it does not find "limits.h"

# ign-msgs
RUN cd ${RL_SOURCES}/ign-msgs &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-transport
RUN cd ${RL_SOURCES}/ign-transport &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-physics
RUN cd ${RL_SOURCES}/ign-physics &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-sensors
RUN cd ${RL_SOURCES}/ign-sensors &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DDRI_TESTS:BOOL=FALSE \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-gui
RUN cd ${RL_SOURCES}/ign-gui &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-fuel-tools
RUN cd ${RL_SOURCES}/ign-fuel-tools &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ign-gazebo
RUN cd ${RL_SOURCES}/ign-gazebo &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

ENV IGN_CONFIG_PATH=${RL_INSTALL}/share/ignition
ENV IGN_GAZEBO_SYSTEM_PLUGIN_PATH=${RL_INSTALL}/lib/ign-gazebo-2/plugins:${RL_INSTALL}/lib/ign-gazebo-2/plugins/gui

# ign-launch
RUN cd ${RL_SOURCES}/ign-launch &&\
    mkdir -p build && cd build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        -DBUILD_TESTING:BOOL=OFF \
        .. &&\
    cmake --build . --target install &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# Fix permissions of the directories inside RL_DIR
RUN find ${RL_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ===========================
# Setup the python virtualenv
# ===========================

ENV VIRTUAL_ENV=/rl/ve
ENV DEVENV_PATH=${VIRTUAL_ENV}/bin:$DEVENV_PATH
RUN pip3 install virtualenv &&\
    virtualenv ${VIRTUAL_ENV} &&\
    ${VIRTUAL_ENV}/bin/pip3 install \
        jupyter \
        docker \
        PyYAML \
        matplotlib \
        pytest \
        gym \
        numpy \
        setuptools_scm \
        cloudpickle \
        # tf-nightly-gpu tfp-nightly \#
        PyVirtualDisplay PILLOW imageio imageio-ffmpeg \
        &&\
    #${VIRTUAL_ENV}/bin/pip3 install git+https://github.com/tensorflow/agents &&\#
    #${VIRTUAL_ENV}/bin/pip3 install --upgrade --force-reinstall tb-nightly &&\#
    ${VIRTUAL_ENV}/bin/pip3 install git+https://github.com/bulletphysics/bullet3.git &&\
    find ${VIRTUAL_ENV} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ===============
# Install PyCharm
# ===============

ARG PYCHARM_VER=2019.2.1
RUN apt-get update &&\
    apt-get install -y --no-install-recommends \
        python-setuptools \
        python3-setuptools \
        libpython-dev \
        libpython3-dev \
        &&\
    rm -rf /var/lib/apt/lists/* &&\
    cd /opt && mkdir pycharm && cd pycharm &&\
    curl -L https://download.jetbrains.com/python/pycharm-community-${PYCHARM_VER}.tar.gz -o /opt/pycharm/installer.tgz &&\
    tar --strip-components=1 -xzf installer.tgz &&\
    rm installer.tgz &&\
    python2 /opt/pycharm/helpers/pydev/setup_cython.py build_ext --inplace &&\
    python3 /opt/pycharm/helpers/pydev/setup_cython.py build_ext --inplace &&\
    ln -s /opt/pycharm/bin/pycharm.sh /usr/local/bin/pycharm

# =============
# Install Julia
# =============

ARG JULIA_VER=1.2.0
ENV DEVENV_PATH=${DEVENV_PATH}:/opt/julia/bin

RUN cd /tmp &&\
    wget https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_VER%.*}/julia-${JULIA_VER}-linux-x86_64.tar.gz &&\
    cd /opt &&\
    tar xvf /tmp/julia-${JULIA_VER}-linux-x86_64.tar.gz &&\
    rm /tmp/julia-${JULIA_VER}-linux-x86_64.tar.gz &&\
    mv julia-${JULIA_VER} julia &&\
    /opt/julia/bin/julia --version

ARG ATOM_VERSION=v1.40.1
ARG ATOM_PKG_TMP=/tmp/atom_packages.txt
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        fakeroot \
        gconf2 \
        gconf-service \
        git \
        gvfs-bin \
        libasound2 \
        libcap2 \
        libgconf-2-4 \
        libgcrypt20 \
        libgtk2.0-0 \
        libgtk-3-0 \
        libnotify4 \
        libnss3 \
        libx11-xcb1 \
        libxkbfile1 \
        libxss1 \
        libxtst6 \
        libgl1-mesa-glx \
        libgl1-mesa-dri \
        policykit-1 \
        python \
        xdg-utils &&\
    rm -rf /var/lib/apt/lists/* &&\
    curl -L https://github.com/atom/atom/releases/download/${ATOM_VERSION}/atom-amd64.deb > /tmp/atom.deb && \
    dpkg -i /tmp/atom.deb && \
    rm -f /tmp/atom.deb && \
    echo "ink" >> ${ATOM_PKG_TMP} &&\ 
    echo "uber-juno" >> ${ATOM_PKG_TMP} &&\ 
    echo "tool-bar" >> ${ATOM_PKG_TMP} &&\
    echo "indent-detective" >> ${ATOM_PKG_TMP} &&\
    echo "latex-completions" >> ${ATOM_PKG_TMP} &&\
    echo "hyperclick" >> ${ATOM_PKG_TMP} &&\ 
    echo "language-julia" >> ${ATOM_PKG_TMP} &&\ 
    echo "julia-client" >> ${ATOM_PKG_TMP} &&\
    apm install --packages-file ${ATOM_PKG_TMP} &&\
    cp -r /root/.atom /opt/dotatom &&\
    find /opt/dotatom -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# TODO: these packages are not available to the user.
# Follow: https://discourse.julialang.org/t/how-does-one-set-up-a-centralized-julia-installation/13922/21
#RUN \
#    /opt/julia/bin/julia -e 'import Pkg; Pkg.add([\
#        "IJulia",\
#        "Plots",\
#        "PyPlot",\
#        "LaTeXStrings",\
#        ]);'

# =======================
# Install projects for RL
# =======================

# Install gym-ignition
RUN ${VIRTUAL_ENV}/bin/pip3 install git+https://github.com/diegoferigo/icub-model-pybullet.git &&\
    apt-get update &&\
    apt install -y --no-install-recommends swig &&\
    rm -rf /var/lib/apt/lists/* &&\
    cd ${RL_SOURCES} &&\
    git clone -b devel https://github.com/robotology/gym-ignition &&\
    mkdir gym-ignition/build &&\
    cd gym-ignition/build &&\
    cmake \
        -G "$CMAKE_GENERATOR" \
        -DCMAKE_BUILD_TYPE=Debug \
        -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
        .. &&\
    cmake --build . --target install &&\
    cd .. &&\
    ${VIRTUAL_ENV}/bin/pip3 install -e . &&\
    find ${RL_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${VIRTUAL_ENV} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# Configure gym-ignition
ENV IGN_FILE_PATH=/iit/local/share:${RL_SOURCES}/icub-gazebo:${RL_INSTALL}/share/gympp/gazebo/models/Minitaur
ENV SDF_PATH=${RL_INSTALL}/share/gympp/gazebo/models
ENV PYTHONPATH=${PYTHONPATH}:${RL_INSTALL}/lib/python3.6/site-packages
ENV ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:${RL_INSTALL}/share/gympp/gazebo

# ========
# iDynTree
# ========

# Use python bindings from fork
RUN cd ${RL_SOURCES} &&\
    git clone -b feature/numpy-and-ik https://github.com/diegoferigo/idyntree &&\
    mkdir -p idyntree/build &&\
    cd idyntree/build &&\
    cmake \
          -GNinja \
          -DCMAKE_BUILD_TYPE=${SOURCES_BUILD_TYPE} \
          -DIDYNTREE_SHARED_LIBRARY:BOOL=OFF \
          -DCMAKE_INSTALL_PREFIX=${RL_INSTALL} \
          -DIDYNTREE_USES_PYTHON:BOOL=ON \
          -DIDYNTREE_USES_PYTHON_VERSION=3.6 \
          -DPYTHON_EXECUTABLE=${VIRTUAL_ENV}/bin/python \
          -DPYTHON_NUMPY_INCLUDE_DIR:PATH=${VIRTUAL_ENV}/lib/python3.6/site-packages/numpy/core/include \
          -DIDYNTREE_USES_KDL:BOOL=OFF \
          -DYCM_USE_DEPRECATED=OFF \
          .. &&\
    cmake --build . --target install &&\
    find ${RL_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+
ENV PYTHONPATH=${RL_INSTALL}/lib/python3.6/site-packages:${PYTHONPATH}

# =============
# FIX OWNERSHIP
# =============

RUN find ${RL_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${RL_INSTALL} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${VIRTUAL_ENV} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# ====================
# Image initialization
# ====================

COPY setup.sh /usr/sbin/setup_rl.sh
COPY entrypoint.sh /usr/sbin/entrypoint_rl.sh
RUN chmod 755 /usr/sbin/setup_rl.sh
RUN chmod 755 /usr/sbin/entrypoint_rl.sh
ENTRYPOINT ["/usr/sbin/entrypoint_rl.sh"]
CMD ["bash"]

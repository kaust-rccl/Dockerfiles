FROM ubuntu:22.04
RUN apt-get update \
&&  DEBIAN_FRONTEND=noninteractive apt-get install -y \
    build-essential \
    gcc-11 g++-11 gfortran-11 curl wget make libnuma-dev libnl-3-200 autoconf \
    libibverbs-dev python3-jinja2 libx11-dev
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 7 \
&&  update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 7 \
&&  update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-11 7

#Install CMake
WORKDIR /software
RUN curl -L https://github.com/Kitware/CMake/releases/download/v3.24.0/cmake-3.24.0-linux-x86_64.sh -o /tmp/cmake-3.24.0-linux-x86_64.sh  && \
       sh /tmp/cmake-3.24.0-linux-x86_64.sh --prefix=/software --skip-license && \
       rm -rf /tmp/*

#Install OpenMPI 
WORKDIR /tmp
RUN curl -o /tmp/openmpi-4.0.3.tar https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.3.tar.gz \
&&  tar xvf /tmp/openmpi-4.0.3.tar 
WORKDIR /tmp/openmpi-4.0.3
RUN ./configure --prefix=/software --with-verbs 
RUN make -j 8 VERBOSE=1
RUN make install
WORKDIR /tmp
RUN rm -rf /tmp/openmpi-4.0.3.tar /tmp/openmpi-4.0.3
RUN ldconfig 
ENV LD_LIBRARY_PATH /software/lib:/usr/local/lib:/usr/lib/x86_64-linux-gnu:${LD_LIBRARY_PATH}
ENV PATH /software/bin:/usr/local/bin:${PATH}

#Install BLAS and LAPACK
WORKDIR /tmp
RUN wget https://github.com/Reference-LAPACK/lapack/archive/refs/tags/v3.11.tar.gz -O lapack-3.11.tar.gz \
&&  tar xvf lapack-3.11.tar.gz 
WORKDIR /tmp/lapack-3.11/build
RUN cmake -DCMAKE_INSTALL_PREFIX=/software -DBUILD_SHARED_LIBS=ON .. \
&&  make -j 4 VERBOSE=1 \
&&  make install 
WORKDIR /tmp
RUN rm -rf /tmp/lapack-3.11.tar.gz /tmp/lapack-3.11 
ENV LD_LIBRARY_PATH /software/lib64:${LD_LIBRARY_PATH}


#Install PETSc
WORKDIR /tmp
RUN wget https://ftp.mcs.anl.gov/pub/petsc/release-snapshots/petsc-3.17.2.tar.gz 
WORKDIR /software
RUN tar xvf /tmp/petsc-3.17.2.tar.gz 
WORKDIR /software/petsc-3.17.2 
ENV PETSC_DIR /software/petsc-3.17.2
ENV PETSC_ARCH=arch-linux-x86_64
RUN ./configure  --with-mpi-dir=/software \
    --with-blaslapack-dir=/software \
    --with-debugging=0 --with-x=0 \
    --with-shared-libraries=0 \
    COPTFLAGS="-fPIC -O3 -std=c99" CXXOPTFLAGS="-fPIC -O3 " FOPTFLAGS="-fPIC -O3" \
&& make PETSC_DIR=/software/petsc-3.17.2 PETSC_ARCH=arch-linux-x86_64 all \
&& make PETSC_DIR=/software/petsc-3.17.2 PETSC_ARCH=arch-linux-x86_64 check

#Install Metis
WORKDIR /tmp
RUN wget https://src.fedoraproject.org/lookaside/pkgs/metis/metis-5.1.0.tar.gz/5465e67079419a69e0116de24fce58fe/metis-5.1.0.tar.gz 
WORKDIR /software
RUN tar xvf /tmp/metis-5.1.0.tar.gz 
WORKDIR /software/metis-5.1.0
RUN sed -i 's;\/\/#define IDXTYPEWIDTH;#define IDXTYPEWIDTH;g' include/metis.h \
&&  sed -i 's;\/\/#define REALTYPEWIDTH;#define REALTYPEWIDTH;g' include/metis.h \
&&  make config \
&&  make 
RUN rm -rf /tmp/*


#Install GTecton
WORKDIR /software
COPY  gtecton.tgz /software
RUN tar xvf gtecton.tgz 
WORKDIR /software/gtecton/build
RUN cmake cmake -DCMAKE_C_COMPILER=mpicc -DCMAKE_Fortran_COMPILER=mpif90  -DMPI_HOME=/software -DMETIS_LIBRARIES=/software/metis-5.1.0/build/Linux-x86_64/libmetis/libmetis.a -DMETIS_INCLUDE_DIR=/software/metis-5.1.0/include .. \
&& make -j 4 VERBOSE=1 \
&& make install
WORKDIR /workspace
RUN rm -rf /software/gtecton.tgz /software/gtecton

ENV PATH /software/bin:${PATH}
ENV LD_LIBRARY_PATH /software/lib:${LD_LIBRARY_PATH}


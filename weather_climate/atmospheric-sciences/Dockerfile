FROM krccl/miniforge:latest
LABEL maintainer="Kadir Akbudak (kadir.akbudak@kaust.edu.sa)"
LABEL version="0.1.0"

RUN mamba create -n atmospheric-sciences -y -c conda-forge wgrib2 eccodes ncl && \
    mamba clean --all --yes

WORKDIR /workdir
COPY entrypoint.sh /software/entrypoint.sh
RUN chmod +x /software/entrypoint.sh
ENTRYPOINT ["/software/entrypoint.sh"]

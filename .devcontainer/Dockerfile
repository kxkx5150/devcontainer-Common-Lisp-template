FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

ARG USERNAME=vscode
ENV DEBIAN_FRONTEND noninteractive


ENV DISPLAY :0
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=C.UTF-8 

# build variables
ARG SBCL_VERSION="2.3.0"
ARG SBCL_URL=https://prdownloads.sourceforge.net/sbcl/sbcl-${SBCL_VERSION}-source.tar.bz2
ARG QUICKLISP_VERSION="2022-04-01"
ARG QUICKLISP_URL=http://beta.quicklisp.org/dist/quicklisp/${QUICKLISP_VERSION}/distinfo.txt


RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
    curl \
    wget \
    git \
    cmake \
    libblas-dev \
    libffi-dev \
    liblapack-dev \
    libz-dev \
    libzmq3-dev \
    rlwrap \
    sbcl \
    gfortran \
    ca-certificates \
    libzstd-dev \
    && apt-get clean -y && rm -rf /var/lib/apt/lists


WORKDIR /src
RUN curl -LO ${SBCL_URL} && \
    tar -xf sbcl-${SBCL_VERSION}-source.tar.bz2 && \
    rm /src/sbcl-${SBCL_VERSION}-source.tar.bz2 && \
    cd /src/sbcl-${SBCL_VERSION} && \
    ln -s /src/sbcl-${SBCL_VERSION} /src/sbcl && \
    bash make.sh --fancy --with-sb-dynamic-core --with-sb-linkable-runtime && \
    (cd src/runtime && make libsbcl.a) && \
    bash install.sh && \
    cd -


USER ${USERNAME}
RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace


# Quicklisp setup
RUN wget -P /tmp/ 'https://beta.quicklisp.org/quicklisp.lisp' \
    && sbcl --noinform --non-interactive --load /tmp/quicklisp.lisp \
    --eval "(quicklisp-quickstart:install :dist-url \"${QUICKLISP_URL}\")" \
    && sbcl --noinform --non-interactive --load /home/${USERNAME}/quicklisp/setup.lisp \
    --eval '(ql-util:without-prompting (ql:add-to-init-file))' \
    && echo '#+quicklisp(push (truename "/usr/src") ql:*local-project-directories*)' >> /home/${USERNAME}/.sbclrc \
    && rm -f /tmp/quicklisp.lisp


USER root
COPY set_uid.sh /home/
ENTRYPOINT ["/home/set_uid.sh"]
COPY ./.inject_bashrc /root
RUN cat /root/.inject_bashrc >> /home/${USERNAME}/.bashrc
CMD ["/bin/sh", "-c", "echo Container started ; trap \"exit 0\" 15; while sleep 1 & wait $!; do :; done"]

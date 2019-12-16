FROM arm32v7/ubuntu
MAINTAINER cb@skeptomai.com

RUN apt-get -y update && \
    apt-get -y install git-core cmake g++ \
    libsdl2-dev libsdl2-image-dev libedit-dev python3 python \
    freeglut3 freeglut3-dev emacs-nox emscripten

RUN git clone https://github.com/WebAssembly/binaryen/ && \
    cd binaryen && \
    cmake . && make && \
    make install

# Additional tools for building wac/wace and wace OS
RUN apt-get -y install python3 nasm xorriso grub-common

COPY node-v13.3.0-linux-armv7l.tar.xz .

RUN xz -d node-v13.3.0-linux-armv7l.tar.xz && \
    tar xvf node-v13.3.0-linux-armv7l.tar

ENV PATH $PATH:/node-v13.3.0-linux-armv7l/bin
RUN npm install -g @kanaka/wamp@1.0.7

FROM ghcr.io/graalvm/graalvm-community:17-ol9 AS builder

WORKDIR /
RUN curl -L -o musl.tar.gz https://more.musl.cc/10/x86_64-linux-musl/x86_64-linux-musl-native.tgz && \
    mkdir musl && tar -xzf musl.tar.gz -C musl --strip-components 1 && \
    rm -f /musl.tar.gz

ENV TOOLCHAIN_DIR="/musl"
ENV PATH="$PATH:${TOOLCHAIN_DIR}/bin"
ENV CC="${TOOLCHAIN_DIR}/bin/gcc"

RUN curl -L -o zlib.tar.gz https://zlib.net/zlib-1.2.13.tar.gz && \
    mkdir zlib && tar -xzf zlib.tar.gz -C zlib --strip-components 1 && cd zlib && \
    ./configure --static --prefix=${TOOLCHAIN_DIR} && \
    make && make install && \
    cd / && rm -rf /zlib && rm -f /zlib.tar.gz

WORKDIR /home/app
COPY classes /home/app/classes
COPY dependency/* /home/app/libs/
COPY *.args /home/app/graalvm-native-image.args
RUN native-image @/home/app/graalvm-native-image.args --static --target=linux-amd64 --libc=musl -H:Class=io.micronaut.build.examples.Application -H:Name=application -cp "/home/app/libs/*:/home/app/classes/"

FROM scratch
COPY --from=builder /home/app/application /app/application
EXPOSE 8080
ENTRYPOINT ["/app/application"]

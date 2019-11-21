# Build stage for Dash Core
FROM alpine as builder

RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories

RUN apk --no-cache --update add \
    autoconf \
    automake \
    cmake \
    curl \
    g++ \
    gcc \
    git \
    libexecinfo-dev \
    libexecinfo-static \
    libtool \
    make \
    perl \
    pkgconfig \
    python3

WORKDIR /build

RUN git clone -b v0.15.0.0 https://github.com/dashpay/dash.git /build

RUN cd depends && make NO_QT=1 -j32

RUN sed -i s:sys/fcntl.h:fcntl.h: src/compat.h
RUN ./autogen.sh

RUN ./configure --prefix=$PWD/depends/x86_64-pc-linux-gnu --without-gui RELDFLAGS="-static --static"

RUN make -j32
RUN strip $PWD/src/dash-cli
RUN strip $PWD/src/dash-tx
RUN strip $PWD/src/dashd

# Build stage for compiled artifacts
FROM alpine
RUN addgroup -S -g 1000 dash && \
    adduser -S -u 1000 -G dash dash
RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add \
  su-exec

ENV DASH_DATA=/home/dash/.dashcore

COPY --from=builder /build/src/dashd /usr/local/bin
COPY --from=builder /build/src/dash-cli /usr/local/bin
COPY --from=builder /build/src/dash-tx /usr/local/bin

COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/dash/.dashcore"]

EXPOSE 9999 9998 19999 19998 19899 19898 19799 19798

ENTRYPOINT ["/entrypoint.sh"]

CMD ["dashd"]

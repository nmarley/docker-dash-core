# Build stage which downloads, verifies and extracts binaries
FROM debian:stable-slim as builder

RUN apt-get update
RUN apt-get install -y wget gnupg

# 63A96B406102E091 : codablock
# 83592BD1400D58D9 : udjinm6
# 4B88269ABD8DF332 : schinzelh
# AAE1CD6BA06E985C : nmarley

RUN set -ex \
  && for key in \
    63A96B406102E091 \
    83592BD1400D58D9 \
    4B88269ABD8DF332 \
    AAE1CD6BA06E985C \
  ; do \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
    gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" ; \
  done

ENV DASH_VERSION=0.15.0.0

WORKDIR /build

RUN wget https://github.com/dashpay/dash/releases/download/v${DASH_VERSION}/SHA256SUMS.asc
RUN wget https://github.com/dashpay/dash/releases/download/v${DASH_VERSION}/dashcore-${DASH_VERSION}-x86_64-linux-gnu.tar.gz
RUN gpg --verify SHA256SUMS.asc
RUN grep " dashcore-${DASH_VERSION}-x86_64-linux-gnu.tar.gz\$" SHA256SUMS.asc | sha256sum -c -
RUN tar -xzf *.tar.gz

# copy Dash binaries to /binaries
RUN mkdir /binaries
RUN cp dashcore-${DASH_VERSION%.*}/bin/dashd /binaries
RUN cp dashcore-${DASH_VERSION%.*}/bin/dash-cli /binaries
RUN cp dashcore-${DASH_VERSION%.*}/bin/dash-tx /binaries


# Build stage
FROM debian:stable-slim

RUN groupadd --gid 1000 dash \
  && useradd --system --uid 1000 --gid 1000 dash \
  && apt-get update -y \
  && apt-get install -y gosu \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV DASH_DATA=/home/dash/.dashcore

COPY --from=builder /binaries/dashd /usr/local/bin
COPY --from=builder /binaries/dash-cli /usr/local/bin
COPY --from=builder /binaries/dash-tx /usr/local/bin

COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/dash/.dashcore"]

EXPOSE 9999 9998 19999 19998 19899 19898 19799 19798

ENTRYPOINT ["/entrypoint.sh"]

CMD ["dashd"]

ARG ENV_REPOSITORY=arcadiacoin/arcadia-env
ARG COMPILER=gcc
FROM ${ENV_REPOSITORY}:${COMPILER}

ARG NETWORK=live
ARG CI_TAG=DEV_BUILD
ARG CI_BUILD=OFF
ADD ./ /tmp/src

WORKDIR /tmp/build

RUN \
cmake /tmp/src \
-DCI_BUILD=${CI_BUILD} \
-DPORTABLE=1 \
-DACTIVE_NETWORK=nano_${NETWORK}_network \
-DNANO_POW_SERVER=ON

RUN \ 
make nano_node -j $(nproc) && \
make nano_rpc -j $(nproc) && \
make nano_pow_server -j $(nproc) && \
cd .. && \
echo ${NETWORK} >/etc/nano-network

FROM ubuntu:20.04

RUN groupadd --gid 1000 arcadia && \
useradd --uid 1000 --gid arcadia --shell /bin/bash --create-home arcadia

COPY --from=0 /tmp/build/nano_node /usr/bin
COPY --from=0 /tmp/build/nano_rpc /usr/bin
COPY --from=0 /tmp/build/nano_pow_server /usr/bin
COPY --from=0 /tmp/src/api/ /usr/bin/api/
COPY --from=0 /etc/nano-network /etc
COPY docker/node/entry.sh /usr/bin/entry.sh
COPY docker/node/config /usr/share/nano/config
RUN chmod +x /usr/bin/entry.sh
RUN mv /usr/bin/nano_node /usr/bin/arcadia_node
RUN ln -s /usr/bin/arcadia_node /usr/bin/rai_node
RUN ln -s /usr/bin/arcadia_node /usr/bin/nano_node
RUN ldconfig

WORKDIR /root
USER root

ENV PATH="${PATH}:/usr/bin"
ENTRYPOINT ["/usr/bin/entry.sh"]
CMD ["arcadia_node", "daemon", "-l"]
ARG REPOSITORY=arcadiacoin/arcadia-node
LABEL org.opencontainers.image.source https://github.com/$REPOSITORY

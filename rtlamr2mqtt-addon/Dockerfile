FROM golang:bullseye as go-builder

WORKDIR /go/src/app

RUN go install github.com/bemasher/rtlamr@latest \
    && apt-get update \
    && apt-get install -y libusb-1.0-0-dev build-essential git cmake \
    && git clone https://github.com/rtlsdrblog/rtl-sdr-blog \
    && cd rtl-sdr-blog \
    && mkdir build && cd build \
    && cmake ../ -DINSTALL_UDEV_RULES=ON \
    && make \
    && make install \
    && cp ../rtl-sdr.rules /etc/udev/rules.d/ \
    && ldconfig

FROM python:3.10-slim

COPY --from=go-builder /usr/local/lib/librtl* /lib/
COPY --from=go-builder /go/bin/rtlamr* /usr/bin/
COPY --from=go-builder /usr/local/bin/rtl* /usr/bin/
COPY ./rtlamr2mqtt.py /usr/bin
COPY ./requirements.txt /tmp
COPY ./sdl_ids.txt /var/lib/

RUN apt-get update \
    && apt-get install -o Dpkg::Options::="--force-confnew" -y \
      libusb-1.0-0 \
    && apt-get --purge autoremove -y \
    && apt-get clean \
    && find /var/lib/apt/lists/ -type f -delete \
    && pip install -r /tmp/requirements.txt \
    && chmod 755 /usr/bin/rtlamr2mqtt.py \
    && rm -rf /usr/share/doc /tmp/requirements.txt

STOPSIGNAL SIGTERM

ENTRYPOINT ["/usr/bin/rtlamr2mqtt.py"]

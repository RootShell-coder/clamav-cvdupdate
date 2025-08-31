FROM python:3.12-slim

RUN apt update && apt install -y --no-install-recommends \
    ca-certificates \
    tini \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /var/cache/apt/* \
    && rm -rf /tmp/* \
    && pip install --no-cache-dir cvdupdate \
    && pip cache purge \
    && useradd -ms /bin/sh clamav \
    && mkdir -p /tmp/cvdupdate-logs && chown -R clamav:clamav /tmp/cvdupdate-logs \
    && mkdir -p /home/clamav/.cvdupdate

COPY config.json state.json /home/clamav/.cvdupdate/
COPY update /usr/local/bin/update

RUN chown -R clamav:clamav /home/clamav \
    && chmod +x /usr/local/bin/update

WORKDIR /home/clamav
USER clamav
ENTRYPOINT ["update"]

# Upstream's Dockerfile (XBEN-042-24/app/Dockerfile) with these changes:
# - the sources are copied from app/app/ (the vendored benchmark folder sits in app/)
# - the build argument FLAG defaults to this lab's fixed flag, the value upstream's Makefile passes
#   (FLAG{sha256 of "XBEN-042-24"}), since the spec passes no build arguments
# - on a Debian buster or bullseye base, apt reads archive.debian.org: both releases have left (or
#   are leaving) the Debian mirrors, so upstream's apt-get install fails there
FROM python:3.8-slim
RUN . /etc/os-release && case "$VERSION_CODENAME" in \
      buster) sec=buster/updates ;; bullseye) sec=bullseye-security ;; *) sec= ;; esac \
 && if [ -n "$sec" ]; then \
      printf 'deb http://archive.debian.org/debian %s main\ndeb http://archive.debian.org/debian-security %s main\n' \
        "$VERSION_CODENAME" "$sec" > /etc/apt/sources.list && rm -f /etc/apt/sources.list.d/*; fi

RUN apt-get update \
    && apt-get install -y --no-install-recommends iputils-ping curl procps openssh-server\
    && rm -rf /var/lib/apt/lists/*

COPY app/app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

WORKDIR /app
COPY app/app/ /app

RUN useradd -m -s /bin/bash pedro && \
    echo 'pedro:AHeavySandbag!' | chpasswd

RUN mkdir /var/run/sshd
RUN echo 'PermitRootLogin no' >> /etc/ssh/sshd_config
RUN echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config

ARG FLAG=FLAG{a0cb3c321396d011884d01c3217fd9400b96561246d29d51f961ada96496d23e}
RUN echo -n $FLAG > /home/pedro/FLAG.txt

COPY app/app/start.sh /start.sh
RUN chmod +x /start.sh

ENTRYPOINT ["/start.sh"]

# CyberCTF per-player flag: the flag above is only the development fallback. ctf-entrypoint.sh
# puts the player's flag (CTF_FLAG_MAIN) in its place at every start, then runs the original
# entrypoint and command, repeated here unchanged.
COPY ctf-entrypoint.sh /usr/local/bin/ctf-entrypoint.sh
ENTRYPOINT ["/usr/local/bin/ctf-entrypoint.sh", "/start.sh"]
CMD []

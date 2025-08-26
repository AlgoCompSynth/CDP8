FROM docker.io/library/ubuntu:24.04
LABEL org.opencontainers.image.authors="znmeb@algocompsynth.com"

ENV DEBIAN_FRONTEND=noninteractive
ENV SOURCE=/usr/local/src
ENV LOGFILES=/usr/local/src/logfiles
ENV CDP8_URL="--branch CDP8.0 https://github.com/ComposersDesktop/CDP8.git"
ENV PORTAUDIO_TARBALL="pa_stable_v190700_20210406.tgz"
ENV PORTAUDIO_URL="https://files.portaudio.com/archives/$PORTAUDIO_TARBALL"

# Linux dependencies and source code
RUN mkdir --parents $LOGFILES
RUN apt-get update > $LOGFILES/update.log 2>&1
RUN apt-get install -qqy --no-install-recommends \
    apt-file \
    build-essential \
    cmake \
    curl \
    file \
    git \
    libasound2-dev \
    libjack-jackd2-dev \
    locales \
    plocate \
    software-properties-common \
    sudo \
    time \
    tree \
    vim \
    wget \
    > $LOGFILES/linux-deps.log 2>&1
RUN apt-file update > $LOGFILES/misc.log 2>&1
RUN updatedb >> $LOGFILES/misc.log 2>&1
WORKDIR $SOURCE
RUN /usr/bin/time git clone $CDP8_URL \
  > $LOGFILES/clone.log 2>&1

# install libaaio
WORKDIR $SOURCE/CDP8/libaaio
RUN tar xf libaaio-0.3.1.tar.bz2 
WORKDIR $SOURCE/CDP8/libaaio/libaaio-0.3.1
RUN ./configure > $LOGFILES/libaaio.log 2>&1
RUN make install >> $LOGFILES/libaaio.log 2>&1

# install portaudio
WORKDIR $SOURCE
RUN wget --quiet $PORTAUDIO_URL > $LOGFILES/portaudio.log 2>&1
RUN tar xf $PORTAUDIO_TARBALL > $LOGFILES/portaudio.log 2>&1
WORKDIR $SOURCE/portaudio
RUN ./configure --with-alsa --with-jack >> $LOGFILES/portaudio.log 2>&1
RUN make install >> $LOGFILES/portaudio.log 2>&1
RUN /sbin/ldconfig --verbose >> $LOGFILES/ldconfig.log 2>&1

# define main user
RUN useradd \
  --comment "CDP8 user" \
  --create-home \
  --groups sudo \
  --shell /bin/bash \
  --skel /etc/skel \
  --user-group \
  CDP8 && \
  echo "CDP8:CDP8" | chpasswd 

# passwordless 'sudo'
RUN echo "CDP8 ALL=(ALL) NOPASSWD: ALL" \
  | tee --append /etc/sudoers.d/cdp8_nopasswd

WORKDIR /home/CDP8
USER CDP8

CMD ["/bin/bash"]

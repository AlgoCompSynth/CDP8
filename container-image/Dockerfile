FROM docker.io/library/ubuntu:noble
LABEL org.opencontainers.image.authors="znmeb@algocompsynth.com"

ENV DEBIAN_FRONTEND=noninteractive
ENV SYS_LOGFILES=/logfiles
ENV CDP8_HOME=/home/CDP8
ENV LOGFILES=$CDP8_HOME/logfiles
ENV CDP8_URL="--branch main https://github.com/ComposersDesktop/CDP8.git"
ENV CDP8_REPO="$CDP8_HOME/CDP8"
#ENV PORTAUDIO_TARBALL="pa_snapshot.tgz"
ENV PORTAUDIO_TARBALL="pa_stable_v190700_20210406.tgz"
ENV PORTAUDIO_URL="https://files.portaudio.com/archives"
ENV PORTAUDIO_PATH="$CDP8_HOME/portaudio"

# Linux dependencies and source code
RUN mkdir --parents $SYS_LOGFILES
RUN apt-get update > $SYS_LOGFILES/update.log 2>&1
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
    > $SYS_LOGFILES/linux-deps.log 2>&1
RUN dpkg-query --list > $SYS_LOGFILES/installed.log 2>&1
RUN apt-file update > $SYS_LOGFILES/misc.log 2>&1
RUN updatedb >> $SYS_LOGFILES/misc.log 2>&1

# define main non-root user
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

WORKDIR $CDP8_HOME
USER CDP8
RUN mkdir --parents $LOGFILES \
  && chown --recursive CDP8:CDP8 $CDP8_HOME

# install portaudio
WORKDIR $CDP8_HOME
RUN wget --quiet $PORTAUDIO_URL/$PORTAUDIO_TARBALL \
  > $LOGFILES/portaudio.log 2>&1
RUN tar xf $PORTAUDIO_TARBALL \
  >> $LOGFILES/portaudio.log 2>&1
WORKDIR $PORTAUDIO_PATH
RUN ./configure --with-alsa --with-jack \
  >> $LOGFILES/portaudio.log 2>&1
RUN make \
  >> $LOGFILES/portaudio.log 2>&1
RUN sudo make install \
  >> $LOGFILES/portaudio.log 2>&1

# clone CDP8
WORKDIR $CDP8_HOME
RUN /usr/bin/time git clone $CDP8_URL \
  > $LOGFILES/clone.log 2>&1

# install libaaio
WORKDIR $CDP8_REPO/libaaio
RUN tar xf libaaio-0.3.1.tar.bz2 \
  > $LOGFILES/libaaio.log 2>&1
WORKDIR $CDP8_REPO/libaaio/libaaio-0.3.1
RUN ./configure \
  >> $LOGFILES/libaaio.log 2>&1
RUN make \
  >> $LOGFILES/libaaio.log 2>&1
RUN sudo make install \
  >> $LOGFILES/libaaio.log 2>&1
RUN sudo /sbin/ldconfig --verbose \
  >> $LOGFILES/libaaio.log 2>&1

# install CDP8
WORKDIR $CDP8_REPO
RUN mkdir build
WORKDIR $CDP8_REPO/build
RUN cmake .. \
  > $LOGFILES/CDP8.log 2>&1
ENV CPATH=$PORTAUDIO_PATH/src/common
ENV VERBOSE=1
RUN make \
  > $LOGFILES/CDP8.log 2>&1
RUN sudo make install \
  > $LOGFILES/CDP8.log 2>&1

WORKDIR $CDP8_HOME

CMD ["/bin/bash"]

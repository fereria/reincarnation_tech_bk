FROM python:3.9-buster

RUN apt-get update
RUN apt-get -y install locales && \
    localedef -f UTF-8 -i ja_JP ja_JP.UTF-8
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
ENV TZ JST-9
ENV TERM xterm

RUN apt-get install -y curl git vim less
RUN apt-get install -y wget libgtk-3-0 libnotify4 libnss3 libxss1 libxtst6 xdg-utils libatspi2.0-0  libsecret-1-0 libgbm-dev libasound2 xvfb sudo
RUN apt-get install libcairo2-dev libfreetype6-dev libffi-dev libjpeg-dev libpng-dev libz-dev

RUN wget http://ftp.us.debian.org/debian/pool/main/libi/libindicator/libindicator3-7_0.5.0-4_amd64.deb
RUN wget http://ftp.us.debian.org/debian/pool/main/liba/libappindicator/libappindicator3-1_0.4.92-7_amd64.deb
RUN sudo apt install -y ./libindicator3-7_0.5.0-4_amd64.deb
RUN sudo apt install -y ./libappindicator3-1_0.4.92-7_amd64.deb

RUN pip install --upgrade pip

RUN DEBIAN_FRONTEND=noninteractive

# mkdocsのインストール
ADD requirements.txt /work_dir/
WORKDIR /work_dir
RUN pip install -r requirements.txt

RUN git config --system --add safe.directory /work_dir
RUN git config --global http.postBuffer 524288000 


EXPOSE 8888
FROM ubuntu:bionic-20210930
# Maintained by
LABEL name="Juicebox tools"
LABEL maintainer="Jason Williams"
LABEL maintainer_email="williams@cshl.edu"
LABEL version="1.0"
#
#
#
USER root
#
#
# Get some necessary packages
RUN apt update -y && apt upgrade -y && apt-get install -y --fix-missing\
 apt-utils\
 apt-transport-https\
 build-essential gcc\
 bzip2\
 ca-certificates\
 curl\
 dialog\
 dirmngr\
 git\
 gnupg2\
 libssl1.0-dev\
 nodejs-dev\
 node-gyp\
 npm nodejs\
 openssl\
 software-properties-common\
 tmux\
 sudo\
 unzip\
 wget\
 zip
#
#
#
# Install conda and add to path via symbolic links
RUN wget https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh &&\
 bash Anaconda3-2021.05-Linux-x86_64.sh -b -p /opt/anaconda3
RUN /opt/anaconda3/bin/conda update -n base -c defaults conda
ENV CONDA_DIR=/opt/anaconda3
RUN sudo ln -s /opt/anaconda3/pkgs/*/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/pkgs/*/lib/* /usr/lib; exit 0
# Configure conda
RUN /opt/anaconda3/bin/conda config --add channels defaults
RUN /opt/anaconda3/bin/conda config --add channels bioconda
RUN /opt/anaconda3/bin/conda config --add channels conda-forge
#
#
# Install iCommands
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - \
    && echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/renci-irods.list\
    && apt-get update \
    && apt-get install -y irods-icommands \
    && apt-get clean \
    && rm -rf /usr/lib/apt/lists/*
#
#
# Install gui/desktop tools
RUN apt update -y && apt upgrade -y && apt-get install -y --fix-missing\
 xauth\
 xserver-xorg-core\
 xserver-xorg\
 xorg\
 openbox
#
#
# Make desktop start at boot
RUN systemctl set-default graphical.target
#
#
#Add the gene user
RUN useradd -p $(openssl passwd -1 -salt xyz 'gene') -m -d /home/gene -s /bin/bash gene
RUN usermod -a -G sudo,root gene
RUN echo "gene ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
#
#
# Switch to GEA user
RUN mkdir -p /home/gene/.irods
RUN chown -R gene:gene /home/gene/.irods
RUN chown gene:gene /home/gene/.bashrc
RUN chown -R gene:gene /opt
#
#
# Install Java and Add Juicebox
RUN apt update -y && apt upgrade -y && apt-get install -y --fix-missing\
 openjdk-11-jdk
COPY Juicebox_1.11.08.jar /home/gene/Juicebox_1.11.08.jar
USER gene
EXPOSE 8888
#
#
# Define entry point and startup commands
ENV HOME /home/gene
CMD /home/gene/Juicebox_1.11.08.jar

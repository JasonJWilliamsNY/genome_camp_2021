FROM ubuntu:bionic-20210930
# Maintained by
LABEL name="Jupyter Lab for Genome Camp"
LABEL maintainer="Jason Williams"
LABEL maintainer_email="williams@cshl.edu"
LABEL version="2.0"
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
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-py37_4.12.0-Linux-x86_64.sh && bash Miniconda3-py37_4.12.0-Linux-x86_64.sh -b -p /opt/anaconda3
RUN /opt/anaconda3/bin/conda update -n base -c defaults conda
ENV CONDA_DIR=/opt/anaconda3
RUN sudo ln -s /opt/anaconda3/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/pkgs/*/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/pkgs/*/lib/* /usr/lib; exit 0
# Configure conda
RUN /opt/anaconda3/bin/conda config --add channels defaults
RUN /opt/anaconda3/bin/conda config --add channels bioconda
RUN /opt/anaconda3/bin/conda config --add channels conda-forge
#
#
# Install iCommands
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" >> /etc/apt/sources.list.d/renci-irods.list && \
    apt-get update && \
    apt install -y irods-icommands
#
#
# Install Jupyter and add Bash kernel
RUN /opt/anaconda3/bin/python3 -m pip install jupyterlab==3.6.0a2
RUN /opt/anaconda3/bin/pip3 install bash_kernel
RUN /opt/anaconda3/bin/python3 -m bash_kernel.install
#
#
## This was not needed before but we need it now
ENV DEBIAN_FRONTEND noninteractive

## Otherwise timedatectl will get called which leads to 'no systemd' inside Docker
ENV TZ UTC
#
#
#Get nodejs upgraded to 12+
RUN curl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -\
  && apt-get update && apt-get install -y nodejs
#
#
#
#
# Add and enable plugins
RUN sudo ln -s /opt/anaconda3/pkgs/*/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/pkgs/*/lib/* /usr/lib; exit 0
RUN /opt/anaconda3/bin/pip3 install jupyterlab_github
RUN /opt/anaconda3/bin/pip3 install jupyterlab_git
RUN /opt/anaconda3/bin/pip3 install jlab-enhanced-cell-toolbar
#
#
#
#Add the gene user
RUN useradd -p $(openssl passwd -1 -salt xyz 'gene') -m -d /home/gene -s /bin/bash gene
#
#
#
# Add sudo to gene user
RUN apt update && \
    apt install -y sudo && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

ARG LOCAL_USER=gene
ARG PRIV_CMDS='/bin/ch*,/bin/cat,/bin/gunzip,/bin/tar,/bin/mkdir,/bin/ps,/bin/mv,/bin/cp,/usr/bin/apt*,/usr/bin/pip*,/bin/yum'

RUN usermod -aG sudo gene && \
    echo "$LOCAL_USER ALL=NOPASSWD: $PRIV_CMDS" >> /etc/sudoers

# Rebuild the Jupyter Lab with new tools
RUN usermod -aG gene gene
#
#
#
# add missing command
RUN apt update -y && apt upgrade -y && apt-get install -y --fix-missing\
 gettext-base\
 nano\
 man
COPY ./jupyter_notebook_config.json /opt/anaconda3/etc/jupyter/jupyter_notebook_config.json
COPY ./jupyter_notebook_config.py /home/gene/.jupyter/jupyter_notebook_config.py
RUN echo ""
RUN chown -R gene:gene /home/gene/.jupyter
COPY .profile /home/gene/.profile
COPY .bashrc /home/gene/.bashrc
RUN chown gene:gene /home/gene/.bashrc
RUN chown gene:gene /home/gene/.profile
#
#
# Switch to gene user
USER gene
ENV PATH="${PATH}:/opt/anaconda3/bin"
WORKDIR /home/gene
EXPOSE 8888
COPY entry.sh /bin

#
#
# Define entry point
ENTRYPOINT ["bash", "/bin/entry.sh"]

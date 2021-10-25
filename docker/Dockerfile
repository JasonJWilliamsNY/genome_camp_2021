FROM ubuntu:bionic-20210930
# Maintained by
LABEL name="Jupyter Lab for Genome Camp"
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
# Install Jupyter and add Bash kernel
RUN /opt/anaconda3/bin/python3 -m pip install jupyterlab==3.2.1
RUN /opt/anaconda3/bin/pip3 install bash_kernel
RUN /opt/anaconda3/bin/python3 -m bash_kernel.install
#
#
#Install R
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		less \
		locales \
    vim-tiny
# Set the locale and environment variables
## Configure default locale, see https://github.com/rocker-org/rocker/issues/19
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& locale-gen en_US.utf8 \
	&& /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

## This was not needed before but we need it now
ENV DEBIAN_FRONTEND noninteractive

## Otherwise timedatectl will get called which leads to 'no systemd' inside Docker
ENV TZ UTC
RUN apt update -qq &&\
 wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc &&\
 add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/" &&\
 apt install -y -q --no-install-recommends r-base r-base-dev
#
#
# More sym links
RUN sudo ln -s /opt/anaconda3/pkgs/*/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/bin/* /bin; exit 0
RUN sudo ln -s /opt/anaconda3/pkgs/*/lib/* /usr/lib; exit 0
#
#
#Install R kernel for jupyter
RUN R -e "install.packages('IRkernel')"
RUN sudo ln -s /opt/anaconda3/bin/* /usr/bin; exit 0
RUN /opt/anaconda3/bin/pip3 install ipywidgets
#RUN apt install nodejs>=12.0
RUN R -e "IRkernel::installspec(user = FALSE)"
#
#
#
#Add the gene user
RUN useradd -p $(openssl passwd -1 -salt xyz 'gene') -m -d /home/gene -s /bin/bash gene
RUN usermod -a -G sudo,root gene
RUN echo "gene ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
#
#
# Switch to GEA user
COPY ./jupyter_notebook_config.json /opt/anaconda3/etc/jupyter/jupyter_notebook_config.json
COPY ./jupyter_notebook_config.py /home/gene/.jupyter/jupyter_notebook_config.py
#COPY ./.bashrc /home/gene/.bashrc
RUN chown -R gene:gene /home/gene/.jupyter
#RUN chmod -R 777 /home/jovyan
RUN chown gene:gene /home/gene/.bashrc
USER gene
#
#
# Define entry point and startup commands
ENTRYPOINT ["jupyter"]
CMD ["lab", "--no-browser", "--ip=0.0.0.0", "--NotebookApp.token=''", "--NotebookApp.password=''", "--notebook-dir='/home/gene'", "--Session.username='gene'"]

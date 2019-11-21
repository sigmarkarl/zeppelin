# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ubuntu:19.04
MAINTAINER Apache Software Foundation <dev@zeppelin.apache.org>

ARG ZEPPELIN_USER_ID=2100
ARG ZEPPELIN_GROUP_ID=2100

# `Z_VERSION` will be updated by `dev/change_zeppelin_version.sh`
ENV Z_VERSION="0.9.0-SNAPSHOT"

ENV LOG_TAG="[ZEPPELIN_${Z_VERSION}]:" \
    Z_HOME="/zeppelin" \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    ZEPPELIN_ADDR="0.0.0.0"

ENV DEBIAN_FRONTEND=noninteractive

RUN groupadd --gid $ZEPPELIN_GROUP_ID zeppelin \
    && useradd -ms /bin/bash -d ${Z_HOME} zeppelin --uid $ZEPPELIN_USER_ID --gid $ZEPPELIN_GROUP_ID

RUN echo "$LOG_TAG update and install basic packages" && \
    apt -y update && \
    apt install -y locales && \
    locale-gen $LANG && \
    apt install -y software-properties-common openjdk-12-jdk && \
    apt -y autoclean && \
    apt -y dist-upgrade && \
    apt-get install -y build-essential

RUN echo "$LOG_TAG install tini related packages" && \
    apt-get install -y wget curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb

# should install conda first before numpy, matploylib since pip and python will be installed by conda
RUN echo "$LOG_TAG Install miniconda2 related packages" && \
    apt-get -y update && \
    apt-get install -y bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    build-essential \
    git mercurial subversion && \
    echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.6.14-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh
ENV PATH /opt/conda/bin:$PATH

RUN echo "$LOG_TAG Install python related packages" && \
    apt-get -y update && \
    apt-get install -y python-dev python-pip && \
    apt-get install -y gfortran && \
    # numerical/algebra packages
    apt-get install -y libblas-dev libatlas-base-dev liblapack-dev && \
    # font, image
    apt-get install -y libpng-dev libfreetype6-dev libxft-dev && \
    # for tkinter
    apt-get install -y python-tk libxml2-dev libxslt-dev zlib1g-dev && \
    pip install numpy pandas matplotlib pandasql ipython jupyter_client ipykernel bokeh ggplot grpcio bkzep parsimonious

RUN echo "$LOG_TAG Install R related packages" && \
    #echo "deb http://cran.rstudio.com/bin/linux/ubuntu bionic-cran35/" | tee -a /etc/apt/sources.list && \

    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu disco-cran35/' && \

    #gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9 && \
    #gpg -a --export E084DAB9 | apt-key add - && \
    apt-get -y update && \
    apt-get -y --allow-unauthenticated install r-base r-base-dev && \
    R -e "install.packages('knitr', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('ggplot2', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('googleVis', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('data.table', repos='http://cran.us.r-project.org')" && \
    # for devtools, Rcpp
    apt-get -y install libcurl4-gnutls-dev libssl-dev && \
    R -e "install.packages('devtools', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('Rcpp', repos='http://cran.us.r-project.org')" && \
    Rscript -e "library('devtools'); library('Rcpp'); install_github('ramnathv/rCharts')"

# Install kubectl
RUN apt update && apt install -y apt-transport-https && \
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list && \
    apt update && apt install -y kubectl

RUN echo "$LOG_TAG Cleanup" && \
    apt-get autoclean && \
    apt-get clean

RUN R -e "if (!requireNamespace('BiocManager', quietly = TRUE)) install.packages('BiocManager')" && \
   R -e "BiocManager::install()" && \
   R -e "install.packages('readxl', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('tidyverse', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('gridExtra', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('sparklyr', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('qqman', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('mctest', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('ggfortify', repos='http://cran.us.r-project.org')" && \
   R -e "install.packages('CMplot', repos='http://cran.us.r-project.org')"

COPY gorr /gorr
RUN R -e "install.packages('/gorr', repos=NULL, type='source')"
RUN rm /usr/bin/python && rm /usr/bin/python3 && ln -s /opt/conda/bin/python3 /usr/bin/python && ln -s /opt/conda/bin/python3 /usr/bin/python3
COPY zeppelin-${Z_VERSION} /zeppelin-${Z_VERSION}
#COPY commons-lang3-3.9.jar /zeppelin-${Z_VERSION}/interpreter/spark/commons-lang3-3.7.jar
#COPY commons-lang3-3.9.jar /zeppelin-${Z_VERSION}/interpreter/spark/commons-lang3-3.8.1.jar
#COPY commons-lang3-3.9.jar /zeppelin-${Z_VERSION}/interpreter/spark/commons-lang3-3.9.jar
#COPY zeppelin-${Z_VERSION}/interpreter/spark/okio-2.2.2.jar /zeppelin-${Z_VERSION}/interpreter/spark/okio-1.6.0.jar
#COPY zeppelin-${Z_VERSION}/interpreter/spark/guice-4.2.2.jar /zeppelin-${Z_VERSION}/interpreter/spark/sisu-guice-3.0.2-no_aop.jar
COPY interpreter-setting.json /zeppelin-${Z_VERSION}/interpreter/spark/interpreter-setting.json

#COPY gor-model-8.1-SNAPSHOT.jar /zeppelin-${Z_VERSION}/interpreter/spark/gor-model-8.1-SNAPSHOT.jar
#COPY gor-gortools-8.1-SNAPSHOT.jar /zeppelin-${Z_VERSION}/interpreter/spark/gor-gortools-8.1-SNAPSHOT.jar

COPY hail-python.zip /zeppelin-${Z_VERSION}/interpreter/spark/pyspark/
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/netty-all-4.0.23.Final.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/jersey-client-1.19.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/jersey-server-1.19.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/jetty-util-6.1.26.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/jetty-6.1.26.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/log4j-over-slf4j-1.7.26.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/log4j-over-slf4j-1.7.28.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/logback-classic-1.2.3.jar
RUN rm -rf zeppelin-${Z_VERSION}/interpreter/spark/logback-core-1.2.3.jar
RUN ln -s /zeppelin-${Z_VERSION} /zeppelin
#COPY gor-scripts/lib ${Z_HOME}/interpreter/spark

#RUN echo "$LOG_TAG Download Zeppelin binary" && \
#    wget -O /tmp/zeppelin-${Z_VERSION}-bin-all.tgz http://archive.apache.org/dist/zeppelin/zeppelin-${Z_VERSION}/zeppelin-${Z_VERSION}-bin-all.tgz && \
#    tar -zxvf /tmp/zeppelin-${Z_VERSION}-bin-all.tgz && \
#    rm -rf /tmp/zeppelin-${Z_VERSION}-bin-all.tgz && \
#    mv /zeppelin-${Z_VERSION}-bin-all/* ${Z_HOME}/ && \
RUN chown -R zeppelin:zeppelin ${Z_HOME}/

COPY log4j.properties ${Z_HOME}/conf/

USER zeppelin

EXPOSE 8080

ENTRYPOINT [ "/usr/bin/tini", "--" ]
WORKDIR ${Z_HOME}
CMD ["bin/zeppelin.sh"]

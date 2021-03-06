FROM ubuntu:bionic

LABEL maintainer="RStudio Docker <docker@rstudio.com>"

# Set versions and platforms
ARG RSP_PLATFORM=xenial
ARG RSP_VERSION=1.4.1106-5
ARG R_VERSION=4.0.2
ARG MINICONDA_VERSION=py37_4.8.3
ARG PYTHON_VERSION=3.7.7
ARG DRIVERS_VERSION=1.7.0
# versions > 3.4.0 require a "v" prefix
ARG CODE_SERVER_VERSION=3.2.0

# Install RStudio Server Pro session components -------------------------------#

RUN apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    curl \
    gdebi \
    libcurl4-gnutls-dev \
    libssl1.0.0 \
    libssl-dev \
    libuser \
    libuser1-dev \
    libpq-dev \
    rrdtool && \
    rm -rf /var/lib/apt/lists/*

RUN curl -O https://s3.amazonaws.com/rstudio-ide-build/session/${RSP_PLATFORM}/rsp-session-${RSP_PLATFORM}-${RSP_VERSION}.tar.gz && \
    mkdir -p /usr/lib/rstudio-server && \
    tar -zxvf ./rsp-session-${RSP_PLATFORM}-${RSP_VERSION}.tar.gz -C /usr/lib/rstudio-server/ && \
    mv /usr/lib/rstudio-server/rsp-session*/* /usr/lib/rstudio-server/ && \
    rm -rf /usr/lib/rstudio-server/rsp-session* && \
    rm -f ./rsp-session-${RSP_PLATFORM}-${RSP_VERSION}.tar.gz && \
    # write session version to a file
    echo "${RSP_VERSION}" > /usr/lib/rstudio-server/SESSION_VERSION

EXPOSE 8788/tcp

# Install additional system packages ------------------------------------------#

RUN apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    git \
    libssl1.0.0 \
    libuser \
    libxml2-dev \
    subversion && \
    rm -rf /var/lib/apt/lists/*

# Install R -------------------------------------------------------------------#

RUN curl -O https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-${R_VERSION}_1_amd64.deb && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive gdebi --non-interactive r-${R_VERSION}_1_amd64.deb && \
    rm -rf r-${R_VERSION}_1_amd64.deb && \
    rm -rf /var/lib/apt/lists/*

RUN ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R && \
    ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript

# Install R packages ----------------------------------------------------------#

RUN /opt/R/${R_VERSION}/bin/R -e 'install.packages("devtools", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")' && \
    /opt/R/${R_VERSION}/bin/R -e 'install.packages("tidyverse", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")' && \
    /opt/R/${R_VERSION}/bin/R -e 'install.packages("shiny", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")' && \
    /opt/R/${R_VERSION}/bin/R -e 'install.packages("rmarkdown", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")' && \
    /opt/R/${R_VERSION}/bin/R -e 'install.packages("plumber", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")'

# Install Python --------------------------------------------------------------#

RUN curl -O https://repo.anaconda.com/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -bp /opt/python/${PYTHON_VERSION} && \
    /opt/python/${PYTHON_VERSION}/bin/pip install virtualenv && \
    rm -rf Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh

ENV PATH="/opt/python/${PYTHON_VERSION}/bin:${PATH}"

# Install Python packages -----------------------------------------------------#

RUN /opt/python/${PYTHON_VERSION}/bin/pip install \
    altair \
    beautifulsoup4 \
    bokeh \
    cloudpickle \
    cython \
    dash \
    dask \
    flask \
    gensim \
    keras \
    matplotlib \
    nltk \
    numpy \
    pandas \
    pillow \
    plotly \
    pyarrow \
    requests \
    scipy \
    scikit-image \
    scikit-learn \
    scrapy \
    seaborn \
    spacy \
    sqlalchemy \
    statsmodels \
    streamlit \
    tensorflow \
    xgboost

# Install Jupyter Notebook and RSP/RSC Notebook Extensions and Packages -------#

RUN /opt/python/${PYTHON_VERSION}/bin/pip install \
    jupyter \
    jupyterlab \
    rsp_jupyter \
    rsconnect_jupyter \
    rsconnect_python

RUN /opt/python/${PYTHON_VERSION}/bin/jupyter-nbextension install --sys-prefix --py rsp_jupyter && \
    /opt/python/${PYTHON_VERSION}/bin/jupyter-nbextension enable --sys-prefix --py rsp_jupyter && \
    /opt/python/${PYTHON_VERSION}/bin/jupyter-nbextension install --sys-prefix --py rsconnect_jupyter && \
    /opt/python/${PYTHON_VERSION}/bin/jupyter-nbextension enable --sys-prefix --py rsconnect_jupyter && \
    /opt/python/${PYTHON_VERSION}/bin/jupyter-serverextension enable --sys-prefix --py rsconnect_jupyter

# Install VSCode code-server --------------------------------------------------#

# TODO: remove the temporary regex to resolve different behavior for code-server
RUN curl -o code-server.tar.gz -L https://github.com/cdr/code-server/releases/download/$( echo $CODE_SERVER_VERSION | sed -r 's/(3\.2\.0)/\1/; t; s/([0-9]\.[0-9]\.[0-9])/v\1/' )/code-server-${CODE_SERVER_VERSION}-linux-amd64.tar.gz && \
    mkdir -p /opt/code-server && \
    tar -zxvf ./code-server.tar.gz -C /opt/code-server/ --strip-components 1 && \
    rm -f ./code-server.tar.gz

# Install RStudio Professional Drivers ----------------------------------------#

RUN apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y unixodbc unixodbc-dev gdebi && \
    rm -rf /var/lib/apt/lists/*

RUN curl -O https://drivers.rstudio.org/7C152C12/installer/rstudio-drivers_${DRIVERS_VERSION}_amd64.deb && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive gdebi --non-interactive rstudio-drivers_${DRIVERS_VERSION}_amd64.deb && \
    rm -rf /var/lib/apt/lists/* && \
    cp /opt/rstudio-drivers/odbcinst.ini.sample /etc/odbcinst.ini

RUN /opt/R/${R_VERSION}/bin/R -e 'install.packages("odbc", repos="https://packagemanager.rstudio.com/cran/__linux__/bionic/latest")'

# Locale configuration --------------------------------------------------------#

RUN apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y locales && \
    rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

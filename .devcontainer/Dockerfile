ARG UBUNTU_VERSION=20.04
ARG CUDA_VERSION=11.4.0
FROM continuumio/miniconda3 AS conda

COPY .devcontainer/env.yml /
RUN conda update conda -c conda-forge && \
    conda env update -f /env.yml -n base && \
    conda clean -afy

FROM nvidia/cuda:${CUDA_VERSION}-runtime-ubuntu${UBUNTU_VERSION}

# Prepare shell and file system
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 SHELL=/bin/bash
ENV PATH /opt/conda/bin:$PATH
SHELL ["/bin/bash", "-c"]

# Install all system stuff, including node
ARG DEBIAN_FRONTEND="noninteractive"
RUN apt-get update && apt-get upgrade -y 
RUN apt-get install -y git build-essential sudo cmake curl wget unzip
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Create the sudo user
ARG USERNAME
RUN useradd $USERNAME -G sudo -s /bin/bash -m && \
    echo $USERNAME' ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER $USERNAME
ENV HOME=/home/$USERNAME
RUN sudo mkdir /ws && sudo chown -R $USERNAME /ws

# Copy over conda and bashrc, install environment
COPY --from=conda --chown=$USERNAME /opt/ /opt/
RUN sudo ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

COPY requirements.txt /tmp
RUN pip install -r /tmp/requirements.txt

# Prepare entrypoint and mount folder
WORKDIR /ws
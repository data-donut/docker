# Global ARGs for username and project directory
# https://docs.docker.com/engine/reference/builder/#arg
ARG USERNAME=data-donut
ARG PROJECT=test
ARG PYTHON_MAJOR=3
ARG PYTHON_MARJOR_MINOR=3.9

#################
# STAGE builder #
#################
# using ubuntu LTS version
FROM ubuntu:22.04

# Renew ARGs from Global ARGs
ARG USERNAME
ARG GROUP_NAME=data-donut
ARG USER_UID=1000
ARG USER_GID=1000
ARG PROJECT
ARG PYTHON_MAJOR
ARG PYTHON_MARJOR_MINOR
# avoid stuck build due to user prompt
ARG DEBIAN_FRONTEND=noninteractive

# Setup ENV for pipenv
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONFAULTHANDLER 1

# Add group and user
RUN groupadd --gid $USER_GID $GROUP_NAME \
    && useradd -m -s /bin/bash --uid $USER_UID --gid $USER_GID --create-home $USERNAME
# To add user to sudo group, run below
# RUN useradd $USERNAME -s /bin/bash -g sudo -m
# -s sets the user's login shell
# -m makes the user's home directory if it doesn't exist: /home/*new-admin-username*
# -g adds the user to the sudo group so they will have admin privileges

# Update apt-get and install necessary softwares
RUN apt-get update && apt-get install -y build-essential procps curl file git

###############################################
# WARNING! Install zsh and sudo for dev only! #
# https://github.com/deluan/zsh-in-docker
RUN apt-get update \
    && apt-get install -y sudo wget \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
# Swtich user from root to $USERNAME to install zsh
USER $USERNAME
# Uses "robbyrussell" theme (original Oh My Zsh theme), with no plugins
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.2/zsh-in-docker.sh)" -- \
    -t robbyrussell
# WARNING! Install zsh and sudo for dev only! #
###############################################

# Minimize homebrew installation time
ENV HOMEBREW_INSTALL_FROM_API=1
# Install homebrew
RUN /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
# Add brew to PATH
ENV PATH="/home/linuxbrew/.linuxbrew/bin:${PATH}"

# Following code are only available in Linux kernal
# DO NOT uncomment them in Docker file
# RUN echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> $HOME/.bash_profile
# RUN eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# Install python
RUN brew install python@$PYTHON_MARJOR_MINOR
# Turns off buffering for easier container logging
ENV PYTHONUNBUFFERED=1
# Soft link python path to /usr/bin
RUN sudo ln -s /home/linuxbrew/.linuxbrew/bin/python$PYTHON_MARJOR_MINOR /usr/bin/python
RUN sudo ln -s /home/linuxbrew/.linuxbrew/bin/python$PYTHON_MARJOR_MINOR /usr/bin/python3
RUN sudo ln -s /home/linuxbrew/.linuxbrew/bin/pip$PYTHON_MARJOR_MINOR /usr/bin/pip
RUN sudo ln -s /home/linuxbrew/.linuxbrew/bin/pip$PYTHON_MARJOR_MINOR /usr/bin/pip3

# Add user's python path
ENV PATH="/home/$USERNAME/.local/bin:${PATH}"
# Install pipenv, 2022.11.30 has permission error
RUN pip install --user pipenv==2022.1.8
# Tell pipenv to create venv in the current directory
ENV PIPENV_VENV_IN_PROJECT=1

WORKDIR /home/$USERNAME/

CMD ["zsh"]

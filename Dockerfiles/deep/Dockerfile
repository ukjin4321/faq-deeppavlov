# Copyright 2017 Neural Networks and Deep Learning lab, MIPT
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM rockylinux/rockylinux:8

ARG COMMIT=master

# Install necessary packages
RUN dnf update -y && \
    dnf install -y \
        epel-release \
        gcc \
        gcc-c++ \
        git \
        make \
        python3 \
        python3-devel \
        which && \
    dnf clean all

# Set locale
RUN localedef -c -f UTF-8 -i en_US en_US.UTF-8
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8

# Install pip and virtualenv
RUN python3 -m ensurepip && \
    python3 -m pip install --no-cache --upgrade pip && \
    python3 -m pip install --no-cache virtualenv==16.7.5

# Set working directory
WORKDIR /base

# Clone DeepPavlov repository and setup
RUN git clone https://github.com/deepmipt/DeepPavlov.git && \
    cd DeepPavlov && \
    git checkout $COMMIT && \
    python3 setup.py develop && \
    python3 -c 'import deeppavlov.models' && \
    rm -rf /root/.cache

# Install additional Python packages
RUN python3 -m pip install email-validator

# Copy configuration and training data
RUN mkdir /base/config
RUN mkdir /base/train
COPY config/tfidf_logreg_en_faq.json /base/config/tfidf_logreg_en_faq.json
COPY train/file.csv /base/train/file.csv

# Create directory for DeepPavlov settings
RUN mkdir -p /root/.deeppavlov

# Install and download DeepPavlov model
RUN python3 -m deeppavlov install /base/config/tfidf_logreg_en_faq.json
RUN python3 -m deeppavlov download /base/config/tfidf_logreg_en_faq.json

# Train DeepPavlov model
RUN python3 -m deeppavlov train /base/config/tfidf_logreg_en_faq.json

# Expose port for API
EXPOSE 5000

# Start DeepPavlov API server
CMD python3 -m deeppavlov riseapi /base/config/tfidf_logreg_en_faq.json
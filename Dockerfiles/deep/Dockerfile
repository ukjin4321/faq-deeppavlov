# Dockerfile

FROM python:3.8-slim-buster

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        libc-dev \
        build-essential \
        libssl-dev \
        libffi-dev \
        git \
        wget \
        curl \
        unzip && \
    rm -rf /var/lib/apt/lists/*

# Install pip
RUN python -m pip install --upgrade pip

# Set working directory
WORKDIR /base

# Install DeepPavlov from GitHub
RUN git clone https://github.com/deepmipt/DeepPavlov.git && \
    cd DeepPavlov && \
    pip install -e .[all]

# Install additional Python packages
RUN pip install email-validator numpy scikit-learn==0.24.2

# Copy configuration and training data
COPY config/tfidf_logreg_en_faq.json /base/config/tfidf_logreg_en_faq.json
COPY train/file.csv /base/train/file.csv

# Create directory for DeepPavlov settings
RUN mkdir -p /root/.deeppavlov

# Metadata setup
ENV ROOT_PATH=/deep
ENV DOWNLOADS_PATH=/deep/downloads
ENV MODELS_PATH=/deep/models

# Download and install DeepPavlov model
RUN python -m deeppavlov install /base/config/tfidf_logreg_en_faq.json
RUN python -m deeppavlov download /base/config/tfidf_logreg_en_faq.json

# Train DeepPavlov model
RUN python -m deeppavlov train /base/config/tfidf_logreg_en_faq.json

# Expose port for API
EXPOSE 5000

# Start DeepPavlov API server
CMD python -m deeppavlov riseapi /base/config/tfidf_logreg_en_faq.json
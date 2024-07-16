# Dockerfile
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

# Install pip and virtualenv
RUN python3 -m ensurepip && \
    python3 -m pip install --no-cache --upgrade pip && \
    python3 -m pip install --no-cache virtualenv==16.7.5

# Set working directory
WORKDIR /base

# Clone DeepPavlov repository and setup
RUN git clone https://github.com/deepmipt/DeepPavlov.git && \
    cd DeepPavlov && \
    git checkout $COMMIT

# Install Python dependencies
RUN cd DeepPavlov && \
    python3 -m pip install --no-cache-dir -r requirements.txt && \
    python3 -m pip install --no-cache-dir -e .

# Install additional Python packages
RUN python3 -m pip install email-validator numpy

# Install scikit-learn compatible version
RUN python3 -m pip install scikit-learn==0.22.2.post1

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
RUN python3 -m deeppavlov install /base/config/tfidf_logreg_en_faq.json
RUN python3 -m deeppavlov download /base/config/tfidf_logreg_en_faq.json

# Train DeepPavlov model
RUN python3 -m deeppavlov train /base/config/tfidf_logreg_en_faq.json

# Expose port for API
EXPOSE 5000

# Start DeepPavlov API server
CMD python3 -m deeppavlov riseapi /base/config/tfidf_logreg_en_faq.json
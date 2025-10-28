FROM nvcr.io/nvidia/cuda:13.0.0-devel-ubuntu24.04
RUN apt update && \
    apt install -y software-properties-common && \
    add-apt-repository -y ppa:ubuntuhandbook1/ffmpeg6 && \
    add-apt-repository ppa:deadsnakes/ppa && \
    find /etc/apt/sources.list.d/ -name "*handbook1*" -exec sed -i 's/noble/focal/g' {} \; && \
    apt update
RUN apt install -y python3.11 \
                    python3.11-dev \
                    python3-dev \
                    build-essential \
                    pkg-config \
                    python3-venv \
                    espeak-ng \
                    espeak-ng-data \
                    git \
                    libsndfile1 \
                    curl \
                    g++ 
RUN apt install -y ffmpeg \
                    libavcodec-dev \
                    libavdevice-dev \
                    libavfilter-dev \
                    libavformat-dev \
                    libavutil-dev \
                    libswresample-dev \
                    libswscale-dev
RUN apt clean && rm -rf /var/lib/apt/lists/* && \
    mkdir -p /usr/share/espeak-ng-data &&  \
    ln -s /usr/lib/*/espeak-ng-data/* /usr/share/espeak-ng-data/ && \
    curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/ && \
    useradd -m -u 1001 appuser && \
    mkdir -p /app/api/src/models/v1_0 && \
    chown -R appuser:appuser /app

USER appuser
WORKDIR /app

# Copy dependency files
COPY --chown=appuser:appuser pyproject.toml ./pyproject.toml

# Install dependencies with GPU extras
RUN uv venv --python 3.10 \
     && uv pip install "Cython>=3.0" \
     && uv pip install "git+https://github.com/PyAV-Org/PyAV.git@v11.0.0" \
     && uv sync --extra gpu --no-cache

# Copy project files including models
COPY --chown=appuser:appuser api ./api
COPY --chown=appuser:appuser web ./web
COPY --chown=appuser:appuser docker/scripts/ ./
RUN chmod +x ./entrypoint.sh


# Set all environment variables in one go
ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app:/app/api \
    UV_LINK_MODE=copy \
    USE_GPU=true \
    PHONEMIZER_ESPEAK_PATH=/usr/bin \
    PHONEMIZER_ESPEAK_DATA=/usr/share/espeak-ng-data \
    ESPEAK_DATA_PATH=/usr/share/espeak-ng-data \
    DEVICE="gpu"

ENV DOWNLOAD_MODEL=false
# Download model if enabled
RUN if [ "$DOWNLOAD_MODEL" = "true" ]; then \
    python download_model.py --output api/src/models/v1_0; \
    fi

# Run FastAPI server through entrypoint.sh
CMD ["./entrypoint.sh"]

# Stage 1: Build virtual environment and compile dependencies
FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /app

# Install build-time dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.10 \
    python3.10-dev \
    python3-pip \
    python3-venv \
    git \
    wget \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create virtual environment
RUN python3.10 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install PyTorch with CUDA 11.8 (matches base image) first
RUN pip install --no-cache-dir \
    torch==2.2.0+cu118 \
    torchvision==0.17.0+cu118 \
    torchaudio==2.2.0+cu118 \
    --index-url https://download.pytorch.org/whl/cu118

# Copy requirements
COPY requirements.txt .

# Install remaining Python dependencies into virtual environment
RUN pip install --no-cache-dir -r requirements.txt


# Stage 2: Runtime environment
FROM nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04 AS runner

ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /app

# Install ONLY runtime system dependencies (no build-tools, no git, no python-dev)
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3.10 \
    ffmpeg \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/* \
    && ln -sf python3.10 /usr/bin/python3 \
    && ln -sf python3 /usr/bin/python

# Copy virtual environment from builder stage
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy application code
COPY . .

# Create necessary directories
RUN mkdir -p /tmp/avatars /tmp/videos /tmp/audio voice_profiles

# Entrypoint: run migrations then start server
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

# Expose port
EXPOSE 8000

# Health check — longer start-period for model loading on first boot
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=5 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

# Run application via entrypoint (handles migrations automatically)
CMD ["./entrypoint.sh"]

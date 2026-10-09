# Slim Debian-based image; the major.minor version is pinned to a release that has
# prebuilt wheels for every dependency in requirements.txt
FROM python:3.12-slim

WORKDIR /src

# System packages needed to build/run the PostgreSQL driver (libpq headers + compiler);
# the apt cache is removed in the same layer to keep the image small
RUN apt-get update -y \
    && apt-get install -y --no-install-recommends build-essential libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first so the dependency layer is cached unless requirements.txt changes
COPY requirements.txt .

# Upgrade packaging tools, then install the Python dependencies without keeping pip's cache
RUN pip install --no-cache-dir --upgrade pip setuptools wheel \
    && pip install --no-cache-dir -r requirements.txt

# Copy the application source (config.py, app.py, ...)
COPY . .

# The Flask app listens on 5153 (APP_PORT overrides it)
EXPOSE 5153

# app.py imports `config`, so it must be started from /src
CMD ["python", "app.py"]

FROM demisto/python3-deb:3.11.9.102626

COPY requirements.txt .

RUN pip install setuptools==57.5.0 && apt-get update && apt-get install -y --no-install-recommends \
  gcc \
  python3-dev \
  git \
&& pip install --no-cache-dir -r requirements.txt \
&& apt-get purge -y --auto-remove \
  gcc git python3-dev \
&& rm -rf /var/lib/apt/lists/*

# One image, two profiles (see .env.example). Built on the operator's arm64 Mac
# and on the Raspberry Pi 5 alike — same architecture, so moving hosts is a
# config change rather than a migration and needs no cross-build step.
FROM python:3.12-slim-bookworm

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# eMudhra/emSign CA compromise (Oct 2026): pip also consults the system root
# store. Distrust emSign ECC Root CA - G3 there explicitly, so the fix does not
# depend on the base image already carrying an updated ca-certificates package.
# No-op once Debian has dropped the root.
RUN sed -i 's|^mozilla/emSign_ECC_Root_CA_-_G3\.crt$|!&|' /etc/ca-certificates.conf \
    && update-ca-certificates --fresh

COPY requirements.txt ./
# pip vendors its own certifi; releases before 26.2.2 still trust the compromised
# eMudhra/emSign root (see requirements.txt). Upgrade pip first so the dependency
# download below already runs over the fixed trust store. Rebuild with
# `docker compose build --pull --no-cache` so neither the base image's system CA
# store nor a cached install layer survives.
RUN pip install --no-cache-dir "pip==26.2.2" \
    && pip install --no-cache-dir -r requirements.txt

COPY . /app/

# Non-root: the container reaches the public internet (Odoo, the LLM provider)
# and shares a box with the operator's other private services.
RUN useradd --system --uid 10001 --home /app appuser \
    && mkdir -p /data/seeds/cache /data/seeds/runs \
    && chown -R appuser:appuser /data
USER appuser

EXPOSE 8000

# Runs from /app so the modules' plain `import config` style keeps working.
CMD ["uvicorn", "web.app:app", "--host", "0.0.0.0", "--port", "8000", \
     "--proxy-headers", "--forwarded-allow-ips", "*"]

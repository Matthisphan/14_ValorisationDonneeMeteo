# Base image configurable pour utiliser une image durcie (DHI) via build-arg.
ARG BACKEND_BASE_IMAGE=python:3.12-slim-bookworm
FROM ${BACKEND_BASE_IMAGE}

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_NO_DEV=1

COPY --from=ghcr.io/astral-sh/uv:0.10.0 /uv /uvx /bin/

COPY . .
# Exec form: evite /bin/sh (absent sur certaines images DHI durcies).
RUN ["/bin/uv", "sync", "--locked"]

EXPOSE 8000
# Serveur WSGI de prod avec logs sur stdout/stderr pour GitHub Actions et Docker logs.
CMD ["/bin/uv", "run", "gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--access-logfile", "-", "--error-logfile", "-", "--log-level", "info"]

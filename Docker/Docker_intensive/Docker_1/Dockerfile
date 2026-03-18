FROM python:3.11
RUN useradd -m appuser
USER appuser
WORKDIR /home/appuser
COPY --chown=appuser:appuser . .
CMD ["python", "app.py"]

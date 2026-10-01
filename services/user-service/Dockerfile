FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN python -m pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

USER 10001:10001

EXPOSE 8000

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import json, urllib.request; r = urllib.request.urlopen('http://127.0.0.1:8000/health', timeout=2); assert r.status == 200 and json.load(r) == {'status': 'ok'}"

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]

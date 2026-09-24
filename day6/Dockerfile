FROM python:3.12-slim

WORKDIR /app

COPY day3/projects/multibranch-ecr-demo/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY day3/projects/multibranch-ecr-demo/app.py .

EXPOSE 5000

CMD ["python", "app.py"]

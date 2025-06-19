FROM python:3.9-buster

WORKDIR /app

COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_RUN_HOST=0.0.0.0

# ... (previous lines remain the same) ...

EXPOSE 8000 

# Command to run your Flask application using Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
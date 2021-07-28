FROM python:3.6-slim

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
RUN mkdir /blogapp
WORKDIR /blogapp
RUN pip install --upgrade pip
COPY requirements.txt /blogapp/

RUN pip install -r requirements.txt
COPY . /blogapp/

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
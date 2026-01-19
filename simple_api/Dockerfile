FROM python:3.13-slim

LABEL maintainer="pozos"

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        python3-dev \
        libsasl2-dev \
        libldap2-dev \
        libssl-dev \
        gcc \
        build-essential && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY student_age.py .
COPY requirements.txt .

RUN pip3 install -r requirements.txt

VOLUME /data

EXPOSE 5000

CMD [ "python3", "./student_age.py" ]

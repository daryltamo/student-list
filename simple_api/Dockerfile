FROM ubuntu:24.04

RUN DEBIAN_FRONTEND=noninteractive apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y python python-pip

RUN pip install flask

COPY . /opt/

ENTRYPOINT FLASK_APP=/opt/student_age.py FLASK_RUN_PORT=8080 flask run --host=0.0.0.0

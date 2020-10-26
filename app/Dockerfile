# basic python image
FROM python:3.7

# install redis to access redis APIs
RUN pip install redis

# Without this setting, Python never prints anything out.
ENV PYTHONUNBUFFERED=1

# declare the source directory
WORKDIR /usr/src/app/

# copy the file
COPY redis_app.py .

# start command
CMD [ "python", "redis_app.py" ]

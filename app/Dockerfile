From python:3.7.5-slim
RUN pip install MarkupSafe==2.0.1 Jinja2==3.0.3 itsdangerous==2.0.1 Werkzeug==2.0.3 flask==1.1.1 redis==3.3.8
WORKDIR /src
COPY ./src/server.py /src/
ENV PORT 80
ENV DEBUG false
ENV REDIS_HOST 127.0.0.1
ENV REDIS_PORT 6379
ENV REDIS_DB 0
ENTRYPOINT ["python", "-u", "server.py"]

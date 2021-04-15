FROM alpine:3.10 as base
  
  RUN echo "**** apk update ****" && \
    apk update && \
    apk add --no-cache bash curl git zip unzip jq

  RUN echo "**** install python3 ****" && \
    apk add --no-cache python3 && \
    if [ ! -e /usr/bin/python ]; then ln -sf python3 /usr/bin/python ; fi && \
    \
    echo "**** install pip and dependent modules ****" && \
    python3 -m ensurepip && \
    rm -r /usr/lib/python*/ensurepip && \
    pip3 install --upgrade pip setuptools && \
    if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi && \
    echo "**** update python and install pip ****" && \
    apk add python-dev alpine-sdk && \
    apk add --virtual build-dependencies && \
    pip install --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org --upgrade pip setuptools && \
    pip install --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org awscli --upgrade --user && \
    mv ~/.local/bin/aws /usr/local/bin && \
    pip install --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org python-gnupg boto3 && \
    python --version && \
    pip --version && \
    aws --version 

  WORKDIR /base
  COPY ./abin ./abin

## Entrypoint access to run python as an interactive shell
FROM python:3.8-slim as runtime
 WORKDIR /app
 COPY --from=base /root/.local /root/.local
 COPY requirements.txt .
 RUN pip install -r requirements.txt
 ENV PATH=/root/.local:$PATH
 # Swapping out the entrypoint between python3 or bash will gain you different access to the container
 # python will give you an interactive Python shell - note: be sure to use python3 as python2 exists for some package dependencies
 # bash will provide an interactive bash shell to the containers system; python and other bins can still be executed as well as any code you copy to the container image
 ENTRYPOINT [ "python3" ]

## Uncomment lines below to run the Flask API sample code and comment out the above runtime image
## Command execution of a sample Flask API
# FROM python:3.8-slim as runtime
#   WORKDIR /app
#   COPY --from=base /root/.local /root/.local
#   COPY ./src .
#   COPY requirements.txt .
#   RUN pip install -r requirements.txt
#   ENV PATH=/root/.local:$PATH
#   EXPOSE 5000
#   CMD [ "python", "./goodbye.py"]
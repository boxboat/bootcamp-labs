# sets the base image for this new Docker image
FROM python:3

# set the DEBIAN_FRONTEND environment variable
ENV DEBIAN_FRONTEND noninteractive

# run a command in the default shell -- install build-essential and wget
RUN apt-get update && apt-get install -y build-essential wget

# run a command in the default shell -- create /root/app/src directory
RUN mkdir -p /root/app/src

# copy files from the host into the Docker image
COPY demo-app.py requirements.txt /root/app/src/

# run a command in the default shell -- install Python dependencies
RUN pip install -r /root/app/src/requirements.txt

# expose port 5000
EXPOSE 5000

# set a healthcheck for the running container
HEALTHCHECK CMD ["wget", "localhost:5000"]

# set the default executable for the container -- cannot be overridden
ENTRYPOINT ["python"]

# set the parameter to pass to the executable specified in ENTRYPOINT
CMD ["/root/app/src/demo-app.py"]

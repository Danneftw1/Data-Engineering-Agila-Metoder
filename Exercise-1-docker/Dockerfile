FROM python:3.9


# Installing required packages with pip
USER root
RUN pip install numpy

# Add the entire src directory to the container at /Exercise1/src
ADD src /Exercise-1-docker/src

# Specify the working directory as /Exercise1 inside the container
WORKDIR /Exercise-1-docker

# Run ex1_0_setup.py when the container starts
CMD ["python", "src/ex_0_setup.py"]
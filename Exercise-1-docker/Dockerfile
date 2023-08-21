# Use the Python 3.9 image
# Sets the base image to Python 3.9
FROM python:3.9

# Set the working directory inside the container to /Exercise1
WORKDIR /Exercise1

# Copy the src folder and Dockerfile into the container at /Exercise1
COPY src/ ./src/
COPY Dockerfile .

# Install the required packages
RUN pipenv install matplotlib plotly_express pandas numpy dash sklearn

# Run ex1_0_setup.py when the container starts
CMD ["python", "src/ex1_0_setup.py"]
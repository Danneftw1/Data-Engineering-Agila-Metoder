FROM python:3.9

WORKDIR /Exercise1

# Add the entire src directory to the container at /Exercise1/src
ADD src /Exercise1/src

# Installing required packages with pip
RUN pip install plotly_express dash scikit-learn

# Run ex1_0_setup.py when the container starts
CMD ["python", "src/ex1_0_setup.py"]


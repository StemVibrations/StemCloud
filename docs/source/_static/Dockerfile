# Use the official Python 3.10 slim image as the base image
FROM python:3.10-slim

# Copy the StemCloud directory from the local filesystem to the container's filesystem
COPY ./StemCloud /StemCloud/

# Copy the requirements.txt file from the local filesystem to the container's filesystem
COPY requirements.txt ./requirements.txt

# Update the package lists for the apt package manager
RUN apt-get -y update

# Install git, which is required for version control and possibly for cloning repositories
RUN apt-get -y install git

# Install Python dependencies specified in the requirements.txt file
RUN pip install -r ./requirements.txt

# Install libgomp1, a runtime library for OpenMP
RUN apt-get install libgomp1

# Install various libraries needed for graphical and other dependencies
RUN apt-get -y install libglu1 libgl1 libxrender1 libxcursor1 libxft-dev libxinerama-dev

# Set the command to run the example script using Python in unbuffered mode
CMD ["python", "-u", "./StemCloud/example/example_stem.py"]

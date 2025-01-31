**Struction app**

![image](https://github.com/user-attachments/assets/3f9ca3c3-5859-4ad0-bba0-aff9a004f82f)

**Write Dockerfile**

    # Use an official Python runtime as a parent image
    FROM python:3.11-slim

    # Set the working directory to /app
    WORKDIR /app

    # Copy the current directory contents into the container at /app
    COPY . /app

    # Install pipenv
    RUN pip install pipenv

    # Install any needed packages specified in Pipfile
    RUN pipenv install
    # --deploy --ignore-pipfile

    EXPOSE 5000 
    # the port here doesnt mean anything.

    # Run flask when the container launches
    CMD ["pipenv", "run", "flask", "--app", "api/app.py", "--debug", "run", "--host=0.0.0.0", "--port=5000"]

# Dice-Project-Part-1-Server
In this part, build Server container using Docker.

# Server Application

This repository contains the server application for serving files and their checksums to clients.

## Setup Instructions

1. **Choose an appropriate base image from the Official Images list.**

2. **Create a Dockerfile for the server container with the following specifications:**

    ```Dockerfile
    # Use an appropriate base image
    FROM python:3.9-slim

    # Set working directory
    WORKDIR /app

    # Copy requirements file
    COPY requirements.txt .

    # Install dependencies
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy server application files
    COPY server.py .

    # Expose port if needed
    EXPOSE 5000

    # Command to run the server application
    CMD ["python", "server.py"]
    ```

3. **Use Docker Compose to define and run the server container:**

    ```yaml
    version: '3'

    services:
      server:
        build: ./server
        ports:
          - "5000:5000"
        volumes:
          - serverdata:/app/serverdata
        networks:
          - app-network

    volumes:
      serverdata:
    ```

4. **Write a server application in your preferred language that does the following:**

    ```python
    import os
    import random
    import hashlib
    from flask import Flask, send_file

    app = Flask(__name__)

    @app.route('/')
    def index():
        return 'This is the server. To get a file, go to /get_file.'

    @app.route('/get_file')
    def get_file():
        file_path = '/app/serverfile.txt'  # Adjusted file path
        file_size = 1024  # 1KB
        random_data = ''.join(random.choices('abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ', k=file_size))
        with open(file_path, 'w') as file:
            file.write(random_data)
        checksum = hashlib.sha256(random_data.encode()).hexdigest()
        return send_file(file_path), 200, {'Checksum': checksum}

    if __name__ == '__main__':
        if not os.path.exists('/app'):
            os.makedirs('/app')
        app.run(host='0.0.0.0', port=5000)
    ```

5. **Ensure the client application is running and accessible at the specified URL (`http://client:5001`).**

6. **Run the server application by building the Docker container and starting it with Docker Compose.**

7. **Pull both the coded in one directory & run the docker compose up**
---

# Client Application

This repository contains the client application for interacting with the server and receiving files.

## Setup Instructions

1. **Choose an appropriate base image from the Official Images list.**

2. **Create a Dockerfile for the client container with the following specifications:**

    ```Dockerfile
    # Use an appropriate base image
    FROM python:3.9-slim

    # Set working directory
    WORKDIR /app

    # Copy requirements file
    COPY requirements.txt .

    # Install dependencies
    RUN pip install --no-cache-dir -r requirements.txt

    # Copy client application files
    COPY client.py .

    # Command to run the client application
    CMD ["python", "client.py"]
    ```

3. **Use Docker Compose to define and run the client container:**

    ```yaml
    version: '3'

    services:
      client:
        build: ./client
        ports:
          - "5001:5001"
        volumes:
          - clientdata:/app/clientdata
        networks:
          - app-network

    volumes:
      clientdata:
    ```

4. **Write a client application in your preferred language that does the following:**

    ```python
    import os
    import requests
    import hashlib

    SERVER_URL = "http://server:5000"  # Updated to use the service name in Docker Compose

    def download_file(url, destination):
        response = requests.get(url)
        with open(destination, 'wb') as file:
            file.write(response.content)

    def calculate_checksum(file_path):
        with open(file_path, 'rb') as file:
            data = file.read()
            return hashlib.sha256(data).hexdigest()

    def main():
        client_data_path = "/app/clientfile.txt"  # Adjusted destination file path
        download_file(f"{SERVER_URL}/get_file", client_data_path)
        checksum = calculate_checksum(client_data_path)
        print(f"Client: File downloaded at {client_data_path}")
        print(f"Client: Checksum: {checksum}")

    if __name__ == "__main__":
        if not os.path.exists('/app/clientdata'):
            os.makedirs('/app/clientdata')
        os.chdir('/app/clientdata')  # Change working directory to '/app/clientdata'
        main()
    ```

5. **Ensure the server is running and accessible at the specified URL (`http://server:5000`).**

6. **Run the client application by building the Docker container and starting it with Docker Compose.**


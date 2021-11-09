# This image already has necessary packages installed, except redis. 
FROM tiangolo/uwsgi-nginx-flask:python3.6
RUN pip install --no-cache-dir redis==3.5.3
# Copy the /azure-vote folder to the /app directory in the container
COPY /azure-vote /app

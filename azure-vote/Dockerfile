# This image already has necessary packages installed, except redis. 
FROM tiangolo/uwsgi-nginx-flask:python3.6
RUN pip install redis
# Copy the /azure-vote folder to the /app directory in the container
ADD /azure-vote /app

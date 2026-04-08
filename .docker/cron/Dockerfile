FROM ubuntu

# Install Cron
RUN apt-get update && apt-get -y install cron

# Copy your cron file into the container
COPY task /etc/cron.d/task

# Give execution rights on the cron job
RUN chmod 0644 /etc/cron.d/task

# Apply cron job
RUN crontab /etc/cron.d/task

# Create the log file to be able to run tail
RUN touch /var/log/cron.log

# Run the command on container startup
CMD ["cron", "-f"]

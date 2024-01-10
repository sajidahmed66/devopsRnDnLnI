# Use an official MySQL image as the base image
FROM mysql:latest

# Set environment variables for MySQL configuration
ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_DATABASE=password

# Copy the SQL dump file into the container's /docker-entrypoint-initdb.d directory
COPY dump.sql ./
# Expose MySQL port
EXPOSE 3306
# CMD ["mysql", "-u root -proot", "use password", "source dump.sql;" ]
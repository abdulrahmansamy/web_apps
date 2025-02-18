# Server setup

### Building the Container Image

1. Create the docker file

```Dockerfile
# Use the UBI 9 Node.js 22 image
FROM registry.access.redhat.com/ubi9/nodejs-22


# Set the working directory
WORKDIR /app

# Create a new user and group
USER root

# Set proper permissions
RUN groupadd -r node && useradd -m -r -g node node \
    && mkdir -p /opt/app-root/src/.npm \
    && chown -R node:node /app && chown -R node:node /opt/app-root/src/.npm

USER node

# Copy package.json and package-lock.json
COPY package*.json app.js ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
# COPY app.js .

# Expose the port the app runs on
EXPOSE 8080

# Command to run the application
CMD ["node", "app.js"]
```

2. build the image

```bash
podman build -f Containerfile -t nodejs-mail-function:latest
```

### Running the container

3. Setting the Environment variables

```bash
echo "your-email@gmail.com" > gmail_user.txt
echo "your-email-password" > gmail_password.txt



podman secret create GMAIL_USER gmail_user.txt
podman secret create GMAIL_PASS gmail_password.txt
```

4. Running the container

```bash
podman build -f Containerfile -t nodejs-mail-function:latest

podman run --name mail-function -p 8080:8080 --rm \
  --secret GMAIL_USER,type=env,target=GMAIL_USER \
  --secret GMAIL_PASS,type=env,target=GMAIL_PASS \
  nodejs-mail-function:latest
```

### Testing

5. Check the server side

```bash
  curl -X POST http://localhost:8080/send-email      -H "Content-Type: application/json"      -H "Authorization: Bearer your_token"      -d '{"name": "John Doe", "email": "john.doe@example.com", "message": "Hello!"}'

```

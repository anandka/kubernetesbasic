# Set the base image to Node
FROM    node

# Install pm2
RUN npm install -g pm2

# Install dependencies
ADD package.json /tmp/package.json

RUN cd /tmp && npm install
RUN mkdir -p /src && cp -a /tmp/node_modules /src/

# Define working directory
WORKDIR /src
ADD . /src

# Expose port
EXPOSE  8080

# Run app using pm2
CMD ["pm2", "start", "/src/index.js", "--no-daemon", "-i", "max"]
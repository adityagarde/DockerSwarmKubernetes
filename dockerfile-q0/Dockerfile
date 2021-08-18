# Specify a base image
FROM node:alpine

WORKDIR /usr/app

# Copy package.json only
COPY ./package.json ./
# Install some depenendencies
RUN npm install

# Copy other files
COPY ./ ./

# Default command
CMD ["npm", "start"]
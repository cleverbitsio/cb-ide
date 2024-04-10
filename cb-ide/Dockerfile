# Use your existing Theia image as the base
FROM terrydhariwal/cb-ide:v1.0

# Install additional dependencies
USER root
RUN apt-get update && apt-get install -y curl python wget vim python3 python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Swtich to Theia user
USER theia
WORKDIR /home/theia/applications/browser

# Launch the backend application via node
ENTRYPOINT [ "node", "/home/theia/applications/browser/lib/backend/main.js" ]

# Arguments passed to the application
CMD [ "/home/project", "--hostname=0.0.0.0" ]
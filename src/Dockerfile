FROM openshift/jenkins-slave:latest

USER root

# Install node version 4.x
RUN curl --silent --location https://rpm.nodesource.com/setup_4.x | bash - \
    && yum -y install nodejs

# Set the npm configuration for cert and set strict ssl false if required
# RUN npm config set strict-ssl false
# RUN npm config set cafile <path to my cert.crt>

# Validate npm install
RUN npm -v

# Install apic toolkit
RUN npm install -g apiconnect
# If you are on-premise, install the apic toolkit from On-Premise API Manager
# RUN npm install -g --unsafe-perm https://apim-hostname/packages/apiconnect

# Validate install
RUN echo n| apic --accept-license

#Set appropriate permissions for .apiconnect directory
RUN chgrp -R 0 /home/jenkins/.apiconnect \
    && chmod -R g=u /home/jenkins/.apiconnect

# Switch to default unelevated user for container process execution
# Do not run container with root user
USER 1001

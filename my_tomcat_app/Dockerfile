FROM rockylinux:8

# Install Java
RUN yum install java -y

# Create and set the working directory for Tomcat
RUN mkdir /opt/tomcat/
WORKDIR /opt/tomcat

# Download and extract Tomcat version 9.0.98
ADD https://downloads.apache.org/tomcat/tomcat-9/v9.0.98/bin/apache-tomcat-9.0.98.tar.gz /opt/tomcat
RUN tar xvfz apache-tomcat-9.0.98.tar.gz && rm -f apache-tomcat-9.0.98.tar.gz
RUN mv apache-tomcat-9.0.98/* /opt/tomcat/

# Expose port 8080
EXPOSE 8080

# Start Tomcat server
CMD ["/opt/tomcat/bin/catalina.sh", "run"]

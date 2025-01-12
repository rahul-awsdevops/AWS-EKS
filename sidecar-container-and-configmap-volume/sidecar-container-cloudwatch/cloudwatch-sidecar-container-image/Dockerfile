# Use the official CloudWatch Agent image as the base image
FROM amazon/cloudwatch-agent:latest

# Copy CloudWatch Agent configuration file (JSON format)
COPY agent.json  /opt/aws/amazon-cloudwatch-agent/bin/default_linux_config.json
ENV RUN_IN_CONTAINER=True
ENTRYPOINT ["/opt/aws/amazon-cloudwatch-agent/bin/start-amazon-cloudwatch-agent"]
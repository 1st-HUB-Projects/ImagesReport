
Streamlit application :

- Lets a user upload an image.
- Extracts GPS coordinates (if available) from the image’s EXIF metadata.
- Uploads the image to Amazon S3.
- Stores the image S3 link and GPS coordinates in a DynamoDB table.
- Finally, you can containerize this Streamlit application with Docker, push it to Amazon ECR, and run it on ECS.

#  Requirements

Python 3.x
Streamlit (for the web UI)
exifread (or you can use Pillow)
Boto3 (AWS SDK for Python)
Your requirements.txt might look like this:

# Grouping Images by Coordinates

Later in your application, you can:

Query the DynamoDB table for all items (images).
Group them by lat/lon.
Present them on a map or cluster them by nearest neighbors.
(Note: If you need more advanced geospatial queries, consider storing lat/lon in a dedicated geospatial index or using additional AWS services like Amazon Location Service.)
# Containerizing the Streamlit App

To run this app in a container on AWS ECS, create a Dockerfile. Below is a simplified example:
```shell
# Use a lightweight Python base image
FROM python:3.9-slim

# Set a working directory
WORKDIR /app

# Copy requirements
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application
COPY . .

# Expose Streamlit's default port
EXPOSE 8501

# Run Streamlit
CMD ["streamlit", "run", "streamlit_app.py", "--server.port=8501", "--server.address=0.0.0.0"]

```

Steps to Deploy on AWS ECS via ECR
Build your Docker image locally or in a CI/CD pipeline:
```shell 
docker build -t my-streamlit-app .
```
Tag your Docker image for ECR:
```shell 
docker tag my-streamlit-app:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-streamlit-app:latest
```

Push the image to ECR:
```shell 
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-streamlit-app:latest
```
Create/Update an ECS Task Definition to reference the newly pushed image in ECR.
Run the task on your ECS cluster (Fargate or EC2 launch type).
Optionally, set up an Application Load Balancer to expose your ECS service to the internet (or keep it private, depending on your use case).



# Final Notes

Make sure your AWS credentials/permissions allow S3 and DynamoDB operations.
In production, you may want to handle edge cases (e.g., no EXIF data, large images, etc.).
Consider using environment variables (e.g., for bucket name, table name) so you can easily configure different environments (dev, staging, prod).
You now have a Streamlit-based image upload workflow that extracts GPS coordinates locally, stores the images in S3, and saves their metadata in DynamoDB.
You can then containerize and deploy the entire app on AWS ECS via ECR.

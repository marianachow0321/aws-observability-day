## About This Fork
   
This fork extends the [aws-samples/one-observability-demo](https://github.com/aws-samples/one-observability-demo) by adding:
- Temporarily fixing the compatibility issue for psycopg and otel exporters

## 1. Rebuild Docker Image for petadoptionshistory-py

The Python-based Pet Adoption History service has been updated and requires rebuilding the Docker image.

### Steps to rebuild the Docker image:

1. Navigate to the petadoptionshistory-py directory:
   ```bash
   cd PetAdoptions/petadoptionshistory-py/
   ```

2. Retrieve the ECR repository
   ```bash
   $AWS_REGION="us-east-2"
   PETHISTORYECR=$(aws ssm get-parameter --name '/petstore/pethistoryrepositoryuri' | jq -r .Parameter.Value)
   aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $PETHISTORYECR
   ```

3. Build and push the Docker image to ECR
   ```bash
      docker build -t pet-adoptions-history:latest .
      docker tag pet-adoptions-history:latest $PETHISTORYECR:latest
      docker push $PETHISTORYECR:latest
   ```

## 2. Redeploy OpenTelemetry Collector Configuration

The OpenTelemetry collector configuration has been updated and needs to be redeployed to your EKS cluster.

### Steps to redeploy the otel-collector-config.yaml:

1. Ensure you have kubectl configured to connect to your EKS cluster:
   ```bash
   aws eks update-kubeconfig --name PetSite --region $AWS_REGION
   kubectl get nodes
   ```

2. Apply the updated OpenTelemetry collector configuration:
   ```bash
   kubectl apply -f PetAdoptions/petadoptionshistory-py/otel-collector-config.yaml
   ```

3. Verify the ConfigMap was updated:
   ```bash
   kubectl get configmap otel-config -n default -o yaml
   ```
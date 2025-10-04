# Docker React App - CI/CD & AWS Deployment

A React application with automated CI/CD pipeline deploying to AWS Elastic Beanstalk using Docker, GitHub Actions, and AWS services.

## 🏗️ Architecture Overview

This project demonstrates a complete CI/CD pipeline that:
- Builds and tests a React application in Docker containers
- Pushes Docker images to Amazon ECR
- Deploys to AWS Elastic Beanstalk with automatic environment management
- Includes proper IAM roles and security configurations

## 🚀 Deployment Pipeline

### GitHub Actions Workflow (`.github/workflows/docker-image.yml`)

The pipeline consists of two main jobs:

#### 1. **Build Job**
- Sets up Docker Buildx for enhanced build capabilities
- Caches Docker layers for faster builds
- Builds the React app using `Dockerfile.dev`
- Runs tests with coverage reporting

#### 2. **Deploy Job**
- Configures AWS credentials using GitHub OIDC
- Pushes Docker images to Amazon ECR
- Creates necessary IAM roles and instance profiles
- Manages Elastic Beanstalk applications and environments
- Handles automatic cleanup of old versions

## 🛠️ Infrastructure Components

### AWS Services Used
- **Amazon ECR**: Docker image registry
- **AWS Elastic Beanstalk**: Application hosting platform
- **Amazon S3**: Deployment artifacts storage
- **IAM**: Security roles and permissions

### Docker Configuration
- **Production Dockerfile**: Multi-stage build with nginx serving
- **Development Dockerfile**: For local development and testing
- **nginx.conf**: Configured for React single-page application routing

## 📁 Project Structure

```
react-app/
├── .github/workflows/
│   └── docker-image.yml          # CI/CD pipeline
├── frontend/
│   ├── Dockerfile                # Production Docker image
│   ├── Dockerfile.dev            # Development Docker image
│   ├── nginx.conf                # Nginx configuration
│   ├── package.json              # React app dependencies
│   └── src/                      # React application source
├── docker-compose.yml            # Local development setup
└── .travis.yml                   # Legacy Travis CI config (deprecated)
```

## 🔧 Setup & Configuration

### Prerequisites
- AWS Account with appropriate permissions
- GitHub repository with GitHub Actions enabled
- AWS CLI configured locally (for manual operations)

### AWS IAM Setup
The pipeline automatically creates:
- **GitHub Actions Role**: `GithubActionsElasticBeanStalkRole`
- **EC2 Instance Role**: `ElasticBeanstalk-EC2-Role`
- **Instance Profile**: `ElasticBeanstalk-EC2-Role`

### Required AWS Policies
- `AWSElasticBeanstalkWebTier`
- `AmazonEC2ContainerRegistryReadOnly`
- Custom policies for GitHub Actions OIDC

## 🚀 Deployment Process

### Automatic Deployment
1. **Trigger**: Push to `main` branch
2. **Build**: Docker image built and tested
3. **Push**: Image pushed to ECR
4. **Deploy**: New version deployed to Elastic Beanstalk
5. **Cleanup**: Old versions automatically removed (keeps last 10)

### Manual Deployment
```bash
# Trigger deployment by pushing changes
git add .
git commit -m "Deploy new version"
git push origin main
```

## 🌐 Accessing the Application

### Finding Your App URL
```bash
# Get environment URL
aws elasticbeanstalk describe-environments \
  --application-name docker-react \
  --environment-names production \
  --query "Environments[0].CNAME" \
  --output text \
  --region ap-southeast-1 \
  --profile personal-profile
```

### Expected URL Format
```
http://production.xyz123.ap-southeast-1.elasticbeanstalk.com
```

## 🔍 Monitoring & Troubleshooting

### Environment Status
```bash
# Check environment health
aws elasticbeanstalk describe-environments \
  --application-name docker-react \
  --query "Environments[*].{Name:EnvironmentName,Status:Status,Health:Health}" \
  --output table \
  --region ap-southeast-1
```

### Common Issues
1. **Environment Terminated**: Usually due to missing IAM permissions (fixed with instance profile)
2. **Build Failures**: Check GitHub Actions logs for Docker build errors
3. **ECR Push Failures**: Verify AWS credentials and ECR repository permissions

### Logs
- **GitHub Actions**: Check workflow runs in GitHub repository
- **Elastic Beanstalk**: View logs in AWS Console under EB environment
- **ECR**: Check image push logs in GitHub Actions

## 🔧 Local Development

### Using Docker Compose
```bash
# Start development environment
docker-compose up

# Run tests
docker-compose run tests
```

### Direct Docker Commands
```bash
# Build development image
docker build -f frontend/Dockerfile.dev -t react-app-dev ./frontend

# Run with tests
docker run -e CI=true react-app-dev npm run test -- --coverage
```

## 📊 CI/CD Features

### Automated Testing
- Runs on every push and pull request
- Uses `CI=true` environment for headless testing
- Generates coverage reports

### Docker Optimization
- Multi-stage builds for smaller production images
- Layer caching for faster builds
- Development and production configurations

### AWS Integration
- OIDC authentication (no long-term AWS keys)
- Automatic resource creation and cleanup
- Environment health monitoring

## 🔐 Security Features

- **GitHub OIDC**: No AWS access keys stored in GitHub
- **Least Privilege**: IAM roles with minimal required permissions
- **Automatic Cleanup**: Old application versions removed automatically
- **Secure Docker**: Multi-stage builds with minimal attack surface

## 📈 Scaling & Performance

### Elastic Beanstalk Auto Scaling
- Automatically scales based on traffic
- Health checks ensure application availability
- Load balancing across multiple instances

### Docker Optimization
- Nginx serving static files efficiently
- Compressed React build for fast loading
- Proper HTTP headers and caching

## 🛠️ Maintenance

### Updating Dependencies
1. Update `package.json`
2. Commit and push changes
3. CI/CD pipeline automatically builds and deploys new version

### Environment Management
- Environments automatically created/updated
- Old versions cleaned up (keeps last 10)
- Health monitoring and automatic recovery

## 📚 Additional Resources

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [React Deployment Guide](https://create-react-app.dev/docs/deployment/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Push to trigger CI/CD pipeline
5. Create a pull request

The CI/CD pipeline will automatically test your changes before deployment.

---

**Note**: This project focuses on CI/CD and deployment practices rather than React development. The React app serves as a demonstration vehicle for the deployment pipeline.

Creating a Docker Compose setup for RStudio Workbench with SSO (Single Sign-On) can be quite complex because it involves multiple components and services. Here is an example of how you might structure such a Docker Compose file. This setup will include:

1. **RStudio Server**: The RStudio Workbench server.
2. **OAuth 2.0 Authentication**: Use an OAuth 2.0 provider for SSO.
3. **User Provisioning**: Configuration for user provisioning.

Below is a simplified example of how you might set this up using Docker Compose. This example assumes you have an OAuth 2.0 provider configuration ready.

### Dockerfile for RStudio Workbench with OAuth 2.0
First, create a `Dockerfile` for the RStudio Workbench that installs necessary R packages and configures authentication.

```dockerfile
# Dockerfile for RStudio Workbench with OAuth 2.0
FROM rocker/rstudio:latest

# Update and install system packages if necessary
RUN apt-get update && apt-get install -y \
  libcurl4-openssl-dev \
  libssl-dev

# Install R packages for OAuth
RUN R -e "install.packages(c('httr', 'httpuv', 'jsonlite'), repos='https://cloud.r-project.org/')"

EXPOSE 8787

# Copy your OAuth configuration script if necessary
COPY oauth2_config.R /etc/rstudio/oauth2_config.R

# Start RStudio Workbench
CMD ["/init"]
```

### Docker Compose File
Create a `docker-compose.yml` file to orchestrate the services.

```yaml
version: '3.9'

services:
  rstudio:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8787:8787"
    environment:
      # Environment variables for RStudio Server
      PASSWORD: "your_password" # Replace with a secure password
      DISABLE_AUTH: "true" # Disable internal authentication for SSO
      OAUTH2_CLIENT_ID: "your_client_id" # Replace with OAuth2 client ID
      OAUTH2_CLIENT_SECRET: "your_client_secret" # Replace with OAuth2 client secret
      OAUTH2_PROVIDER: "your_oauth_provider" # Specify your OAuth 2.0 provider
      OAUTH2_REDIRECT_URI: "http://localhost:8787/auth-callback" # Replace with your redirect URI
    volumes:
      - rstudio-data:/home/rstudio

volumes:
  rstudio-data:
```

### Additional Configuration (Optional)
Depending on the OAuth 2.0 provider, you may need additional configuration files or steps. Here’s a simplified example of an OAuth 2.0 configuration script in R (`oauth2_config.R`).

#### `oauth2_config.R`
```r
library(httr)
library(httpuv)
library(jsonlite)

# Define OAuth 2.0 endpoints
oauth_endpoints <- oauth_endpoint(
  authorize = "https://oauth-provider.example.com/authorize",
  access = "https://oauth-provider.example.com/access_token"
)

# Define your app credentials
my_app <- oauth_app("my_app",
  key = Sys.getenv("OAUTH2_CLIENT_ID"),
  secret = Sys.getenv("OAUTH2_CLIENT_SECRET")
)

# Get OAuth 2.0 token
token <- oauth2.0_token(oauth_endpoints, my_app, scope = "email", cache = FALSE)

# This is a sample code, modify it according to your OAuth 2.0 provider's requirements.
```

#### Steps to Run
1. **Build Docker Image**: Build the Docker image.
   ```sh
   docker-compose build
   ```

2. **Run Docker Compose**: Run the services.
   ```sh
   docker-compose up
   ```

3. **Access RStudio**: Open `http://localhost:8787` in your web browser. You should be redirected to the OAuth 2.0 provider for authentication.

### Important Notes
1. **Security**: Ensure that you use secure methods to handle sensitive information like client secrets and passwords. Use environment variables, Docker secrets, or other secure methods.
2. **Configuration**: Adjust the OAuth 2.0 provider URLs, endpoints, and application credentials based on your specific provider.
3. **Volumes**: Store persistent data in Docker volumes or bind mounts as required.

You may need additional configuration for your specific use case, especially if your OAuth provider or RStudio Server settings require customization.
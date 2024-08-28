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
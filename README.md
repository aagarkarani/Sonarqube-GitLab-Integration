SonarQube Integration with GitLab CI/CD

This guide provides a step-by-step walkthrough for integrating SonarQube into a GitLab CI/CD pipeline. It ensures static code analysis and security scanning is embedded into your DevSecOps workflow.

Overview

This CI pipeline automates the integration of SonarQube analysis into GitLab. It supports token-based authentication using Vault (or your secret management solution), dynamic analysis options via variables, and generates SAST reports consumable by GitLab’s security dashboard.

Prerequisites

GitLab Runner with secret management (Vault or other) integration enabled

SonarQube server accessible from GitLab runners

A configured secret in your secrets engine containing the SonarQube token

Java/Maven project

Secret Injection Example

Use the .fetch_sonarqube YAML template to securely inject the SONAR_TOKEN into your jobs. This template retrieves the token from a secrets manager using GitLab’s ID token mechanism.

.fetch_sonarqube: &fetch_sonarqube
  id_tokens:
    VAULT_ID_TOKEN:
      aud: gitlab-cd-sonarqube
  secrets:
    SONAR_TOKEN:
      token: $VAULT_ID_TOKEN
      file: false
      vault:
        engine:
          name: kv-v1
          path: kv_secrets
        path: shared/sonarqube_token
        field: token

Replace the vault path with your own secret location or inject SONAR_TOKEN directly via GitLab CI/CD settings.

Required CI/CD Variables

Below are the key variables to configure in your GitLab project or group:

SONAR_TOKEN:                 # Required - Injected from Vault or GitLab Settings
SONAR_MAVEN_IMAGE:          # Required - e.g., maven:3.8.6-openjdk-17
SONAR_PROJECT_NAME:         # Required - e.g., my-service
SONAR_SOURCE_DIR:           # Required - e.g., src
SONAR_HOST_URL:             # Required - e.g., https://sonarqube.example.com
SONAR_POM_FILE:             # Optional - Default: pom.xml
SONAR_SETTINGS_FILE:        # Optional - e.g., .m2/settings.xml
SONAR_TIMEOUT:              # Optional - e.g., 300
SONAR_EXCLUSIONS:           # Optional - e.g., **/test/**,**/docs/**
SONAR_CI_NAME:              # Optional - e.g., gitlab-ci-pipeline
SECURITY_DASHBOARD_SOURCE: # Optional - Sonar / None

Pipeline Job Descriptions

1. SonarQube Scan

Runs static code analysis using Maven. Uses the Vault token and dynamic environment variables.
Produces:

- report-task.txt
- sonar-task-id.txt

2. Quality Gate Check

Fetches and verifies Quality Gate status using the CE Task ID. Fails the pipeline if gate fails.
Produces:

- analysis-id.txt

3. SonarQube → GitLab Security Dashboard

Converts SonarQube vulnerabilities (excluding code smells) to GitLab SAST format.
Produces:

- gl-sast-report.json

4. Vulnerability Artifacts (Optional)

Exports findings to JSON format without GitLab dashboard upload.
Produces:

- gl-sonar-report.json

Artifacts Stored Per Job

report-task.txt         # Scanner metadata
sonar-task-id.txt       # Task ID for CE API
analysis-id.txt         # Analysis UUID for results
gl-sonar-report.json    # Full raw issues from SonarQube
gl-sast-report.json     # GitLab-compliant SAST format

Troubleshooting Tips

Confirm secret/token injection is correctly configured for the GitLab runner

Check the SONAR_TOKEN path, permissions, and token validity

Ensure SONAR_PROJECT_NAME matches the SonarQube project key

Inspect logs to confirm variable overrides are applied correctly

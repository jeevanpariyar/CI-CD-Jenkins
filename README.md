# Jenkins Installation on Google Cloud Platform (GCP)

A hands-on project documenting the installation and initial
configuration of **Jenkins** on an **Ubuntu 24.04 LTS** virtual machine
hosted on Google Cloud Platform.

The goal of this project is to prepare a cloud-hosted Jenkins server as
the foundation for learning and building CI/CD pipelines.

## Project Overview

  Item                   Details
  ---------------------- -------------------------------------------
  Cloud provider         Google Cloud Platform (GCP)
  Virtual machine OS     Ubuntu 24.04 LTS
  Automation server      Jenkins
  Java runtime           OpenJDK 21
  Default Jenkins port   TCP 8080
  Access method          Browser over HTTP (for initial lab setup)

> **Important:** Jenkins should not be exposed to the public internet
> without appropriate security controls. Restrict access to trusted
> source IP addresses and use HTTPS and a secure authentication
> configuration for longer-lived environments.

## What is CI/CD?

CI/CD is a set of practices that automates the process of building,
testing, and delivering software.

### Continuous Integration (CI)

When a developer pushes code to a source repository such as GitHub, an
automated pipeline can:

1.  Detect or receive the code change.
2.  Check out the latest source code.
3.  Compile or build the application.
4.  Run automated tests and quality checks.
5.  Report whether the build succeeded or failed.

The objective is to identify integration problems early and make code
changes easier to validate.

### Continuous Delivery and Continuous Deployment (CD)

After the CI stages pass, the pipeline can package and release the
application.

-   **Continuous Delivery:** The application is automatically built,
    tested, and prepared for release. A person may approve the
    production release.
-   **Continuous Deployment:** Every change that passes the required
    checks is automatically deployed to production.

A Jenkins pipeline describes these stages as code, commonly in a file
named `Jenkinsfile`.

## Architecture and Workflow

``` text
Developer
   |
   | Pushes code
   v
GitHub Repository
   |
   | Webhook or pipeline trigger
   v
Jenkins on GCP (Ubuntu 24.04 VM)
   |
   +--> Checkout source code
   |
   +--> Build application
   |
   +--> Run automated tests
   |
   +--> Package application
   |
   +--> Deploy to target environment
```

The deployment stages above describe the planned pipeline workflow. They
are not claimed as completed in this installation project.

## Prerequisites

-   A Google Cloud project with permission to create Compute Engine
    virtual machines.
-   An Ubuntu 24.04 LTS VM.
-   SSH access to the VM.
-   Sufficient disk space for Jenkins, plugins, build tools, and build
    artifacts.
-   A firewall rule that allows TCP port `8080` only from trusted IP
    addresses for initial access.
-   `sudo` privileges on the VM.

## Installation Steps

Run the commands from an SSH session connected to the Ubuntu VM.

### 1. Update Ubuntu package information

``` bash
sudo apt update
```

### 2. Install Fontconfig and OpenJDK 21

Jenkins requires a supported Java runtime. This setup uses OpenJDK 21.

``` bash
sudo apt install -y fontconfig openjdk-21-jre
```

Verify Java:

``` bash
java -version
```

Confirm that the installed Java version is supported by the Jenkins
release you are installing. Check the official Jenkins documentation if
you use a different Jenkins release.

### 3. Add the official Jenkins package repository

Create the keyrings directory:

``` bash
sudo install -d -m 0755 /etc/apt/keyrings
```

Download the Jenkins repository signing key using the current key URL
shown in the official Jenkins installation documentation:

``` bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Add the Jenkins repository:

``` bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" \
  | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

> **Repository key note:** The signing-key filename can change. If the
> URL above is unavailable, do not import a key from an unverified
> source. Follow the current instructions at [Jenkins -- Linux
> installation](https://www.jenkins.io/doc/book/installing/linux/).

### 4. Install Jenkins

Refresh package information and install Jenkins:

``` bash
sudo apt update
sudo apt install -y jenkins
```

### 5. Enable and start the Jenkins service

``` bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check the service status:

``` bash
sudo systemctl status jenkins
```

The expected status is:

``` text
Active: active (running)
```

If Jenkins does not start, inspect the service logs:

``` bash
sudo journalctl -u jenkins -n 100 --no-pager
```

### 6. Check that Jenkins is listening on port 8080

``` bash
sudo ss -lntp | grep 8080
```

Jenkins normally uses TCP port `8080` by default.

### 7. Configure the GCP firewall

In the Google Cloud Console, create or update an ingress firewall rule
for the VM:

-   Direction: Ingress
-   Protocol: TCP
-   Port: `8080`
-   Source: Your trusted public IP address in CIDR notation, such as
    `203.0.113.10/32` (replace this example with your own IP)
-   Target: Only the Jenkins VM, preferably using a network tag

Do not use `0.0.0.0/0` as the source for an internet-accessible Jenkins
server.

### 8. Open Jenkins in a browser

Find the VM's external IP address in the Google Cloud Console, then
open:

``` text
http://YOUR_VM_EXTERNAL_IP:8080
```

Replace `YOUR_VM_EXTERNAL_IP` with the actual external IP address of
your VM.

### 9. Retrieve the initial administrator password

Run this command on the VM:

``` bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password into the Jenkins setup page.

Then:

1.  Select **Install suggested plugins**.
2.  Wait for the plugins to finish installing.
3.  Create your first Jenkins administrator user.
4.  Complete the setup wizard.
5.  Sign in to the Jenkins dashboard.

Never publish the initial administrator password or add it to Git.

## Useful Jenkins and Linux Commands

  ------------------------------------------------------------------------------------
  Task                                Command
  ----------------------------------- ------------------------------------------------
  Check Jenkins status                `sudo systemctl status jenkins`

  Start Jenkins                       `sudo systemctl start jenkins`

  Stop Jenkins                        `sudo systemctl stop jenkins`

  Restart Jenkins                     `sudo systemctl restart jenkins`

  Enable at boot                      `sudo systemctl enable jenkins`

  Disable at boot                     `sudo systemctl disable jenkins`

  View recent Jenkins logs            `sudo journalctl -u jenkins -n 100 --no-pager`

  Follow Jenkins logs                 `sudo journalctl -u jenkins -f`

  Check port 8080                     `sudo ss -lntp \| grep 8080`

  Check Java version                  `java -version`
  ------------------------------------------------------------------------------------

## Troubleshooting

### Jenkins service not found

If you see:

``` text
Unit jenkins.service could not be found.
```

Jenkins may not be installed, or its package installation may have
failed.

Check the package:

``` bash
dpkg -l | grep jenkins
```

If it is not installed, review the repository configuration, run
`sudo apt update`, and install Jenkins.

### Jenkins service fails to start

Check the service logs:

``` bash
sudo journalctl -u jenkins -n 100 --no-pager
```

Also verify Java:

``` bash
java -version
```

Confirm that the installed Jenkins release supports the Java version in
use.

### Jenkins page does not open

Check the following:

1.  Jenkins service is running.
2.  Jenkins is listening on port `8080`.
3.  The VM has an external IP address if accessing it over the public
    internet.
4.  The GCP firewall rule allows TCP `8080` from your current trusted
    public IP.
5.  The VM's operating-system firewall, if enabled, permits the
    connection.

## Key Learnings

-   Updated Ubuntu package repositories using APT.
-   Installed and verified OpenJDK 21.
-   Added a signed Jenkins package repository.
-   Installed Jenkins on an Ubuntu cloud VM.
-   Managed a Linux service using `systemctl`.
-   Checked service status and logs using `systemctl` and `journalctl`.
-   Learned how GCP firewall rules control inbound application access.
-   Prepared the foundation for CI/CD automation.

## Next Steps

The next phase of this learning project is to build an actual CI/CD
pipeline:

1.  Integrate Jenkins with a GitHub repository.
2.  Configure authentication using a narrowly scoped GitHub credential.
3.  Create a `Jenkinsfile` with a simple pipeline.
4.  Automate application builds and tests.
5.  Build and tag a Docker image.
6.  Publish the image to a container registry.
7.  Deploy the application to a test environment.
8.  Add pipeline notifications and build-status reporting.

Start with a simple `Hello World` pipeline, then extend it to a small
Java or Python application.

## Security and Operations Notes

-   Restrict Jenkins access to trusted IP addresses.
-   Do not expose the Jenkins dashboard directly to the entire internet.
-   Use HTTPS through a properly configured reverse proxy or secure
    access layer for persistent environments.
-   Use Jenkins credentials storage for secrets; never hard-code tokens
    or passwords in a `Jenkinsfile`.
-   Grant service accounts and cloud identities only the permissions
    they need.
-   Keep Jenkins and its plugins updated, and review plugin permissions
    and maintenance status.
-   Back up Jenkins configuration and important job data.
-   Monitor disk usage. Build workspaces, logs, plugins, and artifacts
    can consume disk space quickly.
-   For production, consider a more robust architecture, access
    controls, backups, and a tested recovery plan.

## References

-   [Jenkins Documentation](https://www.jenkins.io/doc/)
-   [Installing Jenkins on
    Linux](https://www.jenkins.io/doc/book/installing/linux/)
-   [Jenkins Pipeline
    Documentation](https://www.jenkins.io/doc/book/pipeline/)
-   [Google Cloud Compute Engine
    Documentation](https://cloud.google.com/compute/docs)
-   [Google Cloud VPC Firewall
    Rules](https://cloud.google.com/firewall/docs/firewalls)

------------------------------------------------------------------------

**Project status:** Jenkins installation and initial setup foundation on
GCP. GitHub integration and automated build/deployment pipeline are
planned as the next phase.

**Learning by building. Improving by troubleshooting. Growing through
hands-on experience.**

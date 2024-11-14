# Demo DevSecOps


This Jenkins pipeline is a comprehensive CI/CD pipeline designed for building, testing, and deploying a Docker-based application. 
The pipeline integrates various stages of software development, security testing, and deployment, ensuring that the application is both 
functional and secure at each step. Below is an introduction and a breakdown of its key components:

![demo](/img/CiberM4B3.png)


### **Introduction**
This Jenkins pipeline automates the following CI/CD processes for an application:

1. **Install Dependencies**: It uses a Node.js Docker image to install the necessary dependencies for the application.
2. **Security Static Application Security Testing (SAST)**: The pipeline runs parallel security scans to detect vulnerabilities in the source code and dependencies:
    - **Gitleaks** for detecting sensitive information in the Git history.
    - **Semgrep** for static code analysis based on predefined rules.
    - **Snyk** for scanning known vulnerabilities in dependencies.
    - **NPM Audit** for checking vulnerabilities in JavaScript packages.
3. **SonarQube Analysis**: The application is analyzed for code quality and technical debt using SonarQube.
4. **Docker Build**: A Docker image is built using the Dockerfile located in the repository.
5. **Trivy Scan**: This stage scans the built Docker image for known vulnerabilities using Trivy.
6. **Docker Push**: If the build and security scans pass, the Docker image is pushed to a Docker registry (like Docker Hub or a private registry).
7. **Deploy to Kubernetes**: The application is deployed to a Kubernetes cluster using `kubectl` commands and an `deployment.yaml` file.
8. **Security Dynamic Application Security Testing (DAST)**: The OWASP ZAP tool is used to run dynamic security tests on the running application deployed in Kubernetes.

### **Pipeline Breakdown**

#### **1. Install Dependencies**
- This stage runs in a Docker container (`node:16-alpine`) to install project dependencies using `npm install`.

#### **2. Security SAST (Parallel Scans)**
- **Gitleaks-Scan**: A Docker container running Gitleaks scans for sensitive data such as passwords or API keys within the Git history.
- **Semgrep-Scan**: Uses Semgrep for static code analysis, scanning the codebase for known security vulnerabilities or bad practices.
- **Snyk Test**: Runs a Snyk security scan to identify vulnerabilities in `package.json` dependencies.
- **NPMAudit-Scan**: Uses NPM Audit to check for vulnerabilities in dependencies defined in the `package.json` file.

Each of these security scans runs in parallel to optimize time, with reports being stashed for later use.

#### **3. SonarQube Analysis**
- Executes a SonarQube analysis using the Sonar Scanner tool (`sonar-scanner`), which checks the application code for issues such as code quality, security vulnerabilities, and technical debt.

#### **4. Docker Build**
- Builds a Docker image tagged with the repository name and version, which is derived from the `package.json` file and the Git repository.

#### **5. Trivy-Scan**
- Runs a Trivy scan to detect vulnerabilities in the built Docker image. Trivy is a popular open-source scanner that checks for vulnerabilities in container images.

#### **6. Docker Push**
- If the Docker image is successfully built and passes all security scans, the image is pushed to a Docker registry.

#### **7. Deploy to Kubernetes**
- The pipeline deploys the application to a Kubernetes cluster using `kubectl` commands. It copies a `deployment.yaml` file to the target machine and applies it to deploy the application.

#### **8. Security DAST (OWASP ZAP)**
- Runs an OWASP ZAP (Zed Attack Proxy) baseline scan against the running application in the Kubernetes cluster. This stage checks for common security vulnerabilities like SQL injection, XSS, etc., in the live application.

### **Key Features and Tools**
- **Docker**: Utilized extensively to isolate stages and run tools in consistent environments.
- **Security Tools**: Integration of multiple security testing tools, both static (SAST) and dynamic (DAST), such as Gitleaks, 
    Semgrep, Snyk, NPM Audit, Trivy, and OWASP ZAP.
- **SonarQube**: For code quality analysis, ensuring technical debt and issues are tracked.
- **Kubernetes**: Deployment of the application to a Kubernetes environment for scalability and orchestration.
- **Parallel Execution**: Parallel execution of security scans (SAST) to optimize the overall pipeline execution time.

### **Security Considerations**
The pipeline employs a layered security approach:
- **SAST** tools (Gitleaks, Semgrep, Snyk, NPM Audit) scan the source code and dependencies for vulnerabilities.
- **DAST** with OWASP ZAP scans the deployed application for runtime vulnerabilities.
- **Trivy** ensures that the Docker image itself is free from known vulnerabilities before deployment.

### **Conclusion**
This Jenkins pipeline automates a complete CI/CD process, with a strong emphasis on security at both the code and runtime levels. It ensures that only secure and stable Docker images are deployed to Kubernetes, providing confidence in both the functionality and security of the application throughout its lifecycle.



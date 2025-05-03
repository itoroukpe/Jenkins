Here is a **step-by-step guide** to writing Jenkins Pipeline code using the **Jenkins Pipeline Syntax Generator** for integrating:

* **Git** (source control)
* **Maven** (build)
* **SonarQube** (code analysis)
* **Nexus** (artifact deployment)
* **Tomcat** (deployment)

---

### ✅ Prerequisites

Before starting, ensure the following are set up:

* Jenkins is installed and running.
* Required Jenkins plugins:

  * **Pipeline**
  * **Git Plugin**
  * **Pipeline: Maven Integration**
  * **SonarQube Scanner**
  * **Nexus Artifact Uploader**
  * **Deploy to container Plugin**
* Jenkins credentials (for Git, Nexus, Tomcat) are saved.
* SonarQube and Nexus URLs and authentication are configured in Jenkins.

---

### 🚀 Step-by-Step Guide Using Jenkins Syntax Generator

---

#### 1. **Git Checkout**

**Navigate to**: `Jenkins → New Item → Pipeline → Configure → Pipeline Syntax`

* **Select**: `checkout: General SCM`
* **SCM**: Git
* **Repository URL**: `https://github.com/your-org/your-repo.git`
* **Credentials**: Select if private repo
* **Branches to build**: `*/main`

✅ **Click "Generate Pipeline Script"**

```groovy
checkout([$class: 'GitSCM', branches: [[name: '*/main']], 
  userRemoteConfigs: [[url: 'https://github.com/your-org/your-repo.git']]])
```

---

#### 2. **Maven Build**

**Go to**: Pipeline Syntax → `Sample Step`: `sh`

**Command**: `mvn clean package`

✅ Generated:

```groovy
sh 'mvn clean package'
```

---

#### 3. **SonarQube Analysis**

**Go to**: Pipeline Syntax → `Sample Step`: `withSonarQubeEnv`

* **Name**: Select your SonarQube server

✅ Generated:

```groovy
withSonarQubeEnv('MySonarQubeServer') {
    sh 'mvn sonar:sonar'
}
```

---

#### 4. **Upload to Nexus**

Use the `nexusArtifactUploader` step.

**Go to**: Pipeline Syntax → `Sample Step`: `nexusArtifactUploader`

* Fill out:

  * Nexus version
  * Protocol (`http`)
  * Nexus URL
  * Group ID, Artifact ID
  * Version
  * Repository (e.g., releases/snapshots)
  * Credentials

✅ Example:

```groovy
nexusArtifactUploader artifacts: [[artifactId: 'my-app', classifier: '', file: 'target/my-app.jar', type: 'jar']], 
    credentialsId: 'nexus-creds', 
    groupId: 'com.mycompany.app', 
    nexusUrl: 'nexus.mycompany.com', 
    nexusVersion: 'nexus3', 
    protocol: 'http', 
    repository: 'releases', 
    version: '1.0.0'
```

---

#### 5. **Deploy WAR to Tomcat**

**Go to**: Pipeline Syntax → `deploy adapters` if `Deploy to container` plugin is installed.

Use `deploy adapters` or use simple shell:

```groovy
sh 'curl -u tomcatuser:password -T target/my-app.war http://your-tomcat-server:8080/manager/text/deploy?path=/my-app&update=true'
```

---

### 🧩 Final Jenkinsfile Sample

```groovy
pipeline {
    agent any

    environment {
        SONARQUBE = 'MySonarQubeServer'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/your-org/your-repo.git']]])
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'my-app', classifier: '', file: 'target/my-app.jar', type: 'jar']],
                    credentialsId: 'nexus-creds',
                    groupId: 'com.mycompany.app',
                    nexusUrl: 'nexus.mycompany.com',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'releases',
                    version: '1.0.0'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh 'curl -u tomcatuser:password -T target/my-app.war http://your-tomcat-server:8080/manager/text/deploy?path=/my-app&update=true'
            }
        }
    }
}
```

---



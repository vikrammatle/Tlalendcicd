env.PROJECT_GIT_NAME = 'GLOBALBI_POC'
env.PROJECT_NAME = env.PROJECT_GIT_NAME.toLowerCase()
env.JOB = 'test'
env.VERSION = '0.1'
env.GIT_URL = 'https://github.com/vikrammatle/Tlalendcicd.git'
env.TYPE = "" // if big data = _mr
env.DOCKERHUB_USER = "talendinc"
env.imageName= 'talendimage'
env.registry='http://464598779341.dkr.ecr.ap-south-1.amazonaws.com'


// Credentials IDs (Manage Jenkins => Credentials)
env.GIT_CREDENTIALS_ID = 'github'

node {
 	// Clean workspace before doing anything
    try {
        def userInput
        def deployprod
        stage('Initialize') {
            sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
        }
        stage ('Git Checkout') {
            git(
                url: "${GIT_URL}",
                credentialsId: "${GIT_CREDENTIALS_ID}",
                branch: 'master'
            )       
            mvnHome = tool 'mvn'
        }
        stage ('Build, Test and Publish Jobs to Nexus') {
                    withMaven(
                            // Maven installation declared in the Jenkins "Global Tool Configuration"
                            maven: 'mvn',
                            // Maven settings.xml file defined with the Jenkins Config File Provider Plugin
                            // Maven settings and global settings can also be defined in Jenkins Global Tools Configuration
                           // mavenSettingsConfig: 'maven-file',
                            mavenOpts: '-Dupdatesite.path=http://13.233.125.1:8082/P2/ -Dlicense.path=/opt/remote.license -Dgeneration.type=local  -DaltDeploymentRepository=id::default::http://admin:admin123@13.232.167.109:8081/repository/thirdparty/ -Xms1024m -Xmx3096m') 
                            {
                    
                        // Run the maven build
                        sh "mvn -s /opt/settings.xml -f $PROJECT_GIT_NAME/poms/pom.xml clean deploy"
                    
                        }  
        }
         stage ('building the docker image') {
               script{
                    sh "pwd"
                    sh "ls -l"
                    dir("./GLOBALBI_POC") {
                    docker.build(imageName, "-f DockerFile .")
                    }
            }
        }
      	stage("Push to Registry") {
                script {
                    sh "eval \$(/var/lib/jenkins/bin/aws ecr get-login --region ap-south-1 --no-include-email)"
                    sh "/var/lib/jenkins/bin/aws ecr describe-repositories --region ap-south-1 --repository-names $imageName || /var/lib/jenkins/bin/aws ecr create-repository --region ap-south-1 --repository-name $imageName"
                    docker.withRegistry(registry) {
                        docker.image(imageName).push('latest')
                    }
                    //sh "/var/lib/jenkins/bin/aws ecr list-images --region $REGION --repository-name $imageName --filter tagStatus=UNTAGGED --query 'imageIds[*]' --output text | while read imageId; do /var/lib/jenkins/bin/aws ecr batch-delete-image --region $REGION --repository-name $imageName --image-ids imageDigest=\$imageId; done"
                }
        }
  
    } catch (err) {
        currentBuild.result = 'FAILED'
        throw err
    }
}

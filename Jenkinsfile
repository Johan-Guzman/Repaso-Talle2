pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        REGISTRY  = 'localhost:9080'
        NEXUS_MVN = 'http://nexus:8081/repository/maven-releases/'
        API_IMAGE = 'studytrack-api'
        WEB_IMAGE = 'studytrack-frontend'
    }

    stages {

        stage('Checkout & Test') {
            steps {
                checkout scm

                script {
                    env.GIT_SHORT = sh(
                        script: 'git rev-parse --short=7 HEAD',
                        returnStdout: true
                    ).trim()

                    env.VERSION = "1.0.${env.BUILD_NUMBER}-${env.GIT_SHORT}"

                    echo "Version: ${env.VERSION}"
                }

                dir('codigo_base/backend') {
                    sh 'mvn -B test'
                }
            }
        }

        stage('Package & Tag Inmutable') {
            steps {
                dir('codigo_base/backend') {
                    sh "mvn -B versions:set -DnewVersion=${env.VERSION} -DgenerateBackupPoms=false"
                    sh 'mvn -B package -DskipTests'
                }

                sh "docker build -t ${REGISTRY}/${API_IMAGE}:${env.VERSION} codigo_base/backend"
                sh "docker build -t ${REGISTRY}/${WEB_IMAGE}:${env.VERSION} codigo_base/frontend"
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {
                    sh '''
                        cat > settings.xml <<EOF
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASS}</password>
        </server>
    </servers>
</settings>
EOF

                        cd codigo_base/backend

                        mvn -B \
                            -s ../settings.xml \
                            deploy \
                            -DskipTests \
                            -Dnexus.maven.url=${NEXUS_MVN}

                        cd ..

                        rm -f settings.xml

                        echo "$NEXUS_PASS" | docker login ${REGISTRY} \
                            -u "$NEXUS_USER" \
                            --password-stdin

                        docker push ${REGISTRY}/${API_IMAGE}:${VERSION}
                        docker push ${REGISTRY}/${WEB_IMAGE}:${VERSION}

                        docker logout ${REGISTRY}
                    '''
                }
            }
        }

        stage('Deploy & Smoke Test') {
            steps {
                sh "REGISTRY=${REGISTRY} TAG=${env.VERSION} docker compose -f deploy/docker-compose.yml up -d"

                sh '''
                    for i in $(seq 1 15); do

                        if curl -fs http://host.docker.internal:8080/api/tasks > /dev/null; then
                            echo "Smoke test OK"
                            exit 0
                        fi

                        echo "Intento $i fallido, reintentando..."
                        sleep 4
                    done

                    echo "Smoke test FALLO"
                    exit 1
                '''
            }
        }
    }

    post {
        failure {
            echo "Pipeline fallo en version ${env.VERSION}"
        }

        success {
            echo "Pipeline OK: ${env.VERSION}"
        }
    }
}

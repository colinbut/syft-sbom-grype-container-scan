pipeline{
    agent any
    stages{
        stage("Build Jar"){
            steps{
                sh './mvnw clean install -Dcheckstyle.skip=true -Dmaven.test.skip=true'
            }
        }
        stage("Build Container"){
            steps{
                sh 'docker build -t app:${BUILD_NUMBER} .'
            }
        }
        stage("Generate Software Bill of Materials (sbom) with Syft"){
            steps{
                sh '''
                    curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
                    syft app:${BUILD_NUMBER} --scope all-layers -o json > sbom-${BUILD_NUMBER}.json
                    syft app:${BUILD_NUMBER} --scope all-layers -o table > sbom-${BUILD_NUMBER}.txt
                '''
            }
        }
        stage("Generate vulnerability listing with Grype") {
            steps {
                sh 'curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin'
                sh 'grype sbom:sbom-${BUILD_NUMBER}.json'
                sh 'grype sbom:sbom-${BUILD_NUMBER}.json > vulnerabilities-${BUILD_NUMBER}.json'
            }
        }
        stage("Cleanup") {
            steps {
                archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom*,vuln*', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
                sh '''
                    rm -rf sbom* && rm -rf vulner*
                '''
            }
        }
    }
}

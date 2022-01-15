# syft-sbom-grype-container-scan

This example showcases how to integrate software bill of materials (sbom) generation tooling and container vulnerability scanning tooling into a Jenkins Pipeline.

In this particular example I am just simply installing the tools ([syft](https://github.com/anchore/syft) and [grype](https://github.com/anchore/grype)) on the instance running Jenkins and then execute the CLI tool command passing in the container image to analyse with each being in a separate pipeline stage.

For example:

## Generating Software Bill of Materials (sbom) with Syft

```groovy
stage("Generate Software Bill of Materials (sbom) with Syft"){
    steps{
        sh '''
            curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
            syft app:${BUILD_NUMBER} --scope all-layers -o json > sbom-${BUILD_NUMBER}.json
            syft app:${BUILD_NUMBER} --scope all-layers -o table > sbom-${BUILD_NUMBER}.txt
        '''
    }
}
```

## Container Vulnerability Scanning with Grype

```groovy
stage("Generate vulnerability listing with Grype") {
    steps {
        sh 'curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin'
        sh 'grype sbom:sbom-${BUILD_NUMBER}.json'
        sh 'grype sbom:sbom-${BUILD_NUMBER}.json > vulnerabilities-${BUILD_NUMBER}.json'
    }
}
```

Finally, for simplicity sake, the CLI tools output the results in files which i then archive in Jenkins for viewing at my pleasure.

```groovy
archiveArtifacts allowEmptyArchive: true, artifacts: 'sbom*,vuln*', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
```

## Authors

Colin But.

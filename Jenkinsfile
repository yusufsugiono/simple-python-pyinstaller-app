node {
    stage('Build') { 
        withDockerContainer(image: 'python:2-alpine') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
            stash(name: 'compiled-results', includes: 'sources/*.py*')  
        }
    }
    stage('Test') {
        withDockerContainer(image: 'qnib/pytest') {
            try {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            catch (e) {
                throw e
            }
            finally {
                junit 'test-reports/results.xml'
            }
        }
    }
    stage('Deploy') {
        withEnv(['VOLUME=$(pwd)/sources:/src', 'IMAGE=cdrx/pyinstaller-linux:python2']) {
            try{
                dir(path: env.BUILD_ID){
                    unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            catch(e) {
                throw e
            }
            finally {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
            }
        }
    }
}
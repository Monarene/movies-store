def imageName = 'mlabouardy/movies-store'
def myImageName = 'monarene/movie-parser'

node(''){
    stage('Checkout'){
        checkout scm
    }

    def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")

    stage('Tests'){
        parallel(
            'Quality Tests': {
                sh "docker run --rm ${imageName}-test npm run lint"
            },
            'Integration Tests': {
                sh "docker run --rm ${imageName}-test npm run test"
            },
            'Coverage Reports': {
                sh "docker run --rm -v $PWD/coverage:/app/coverage ${imageName}-test npm run coverage-html"
                publishHTML (target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: "$PWD/coverage",
                    reportFiles: "index.html",
                    reportName: "Coverage Report"
                ])
            }
        )
    }

    stage('Build'){
        dockerImage = docker.build(imageName)
    }

    stage('Push'){
        withDockerRegistry([credentialsId: "dockerhub", url: "" ]) {
        dockerImage.push(commitID())
        
        if (env.BRANCH_NAME == 'develop') {
            dockerImage.push('develop')
        }
        
        }
        
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}
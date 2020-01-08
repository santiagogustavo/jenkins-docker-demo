node {
  def commit_id
  stage('Preparation') {
    checkout scm
    sh "git rev-parse --short HEAD > .git/commit-id"
    commit_id = readFile('.git/commit-id').trim()
  }
  stage('test') {
    def myTestContainer = docker.image('node:latest')
    myTestContainer.pull()
    myTestContainer.inside {
      sh 'npm install --only=dev'
      sh 'npm test'
    }
  }
  stage('test with a DB') {
    def mysql = docker.image('mysql:5').run("-e MYSQL_ALLOW_EMPTY_PASSWORD=yes --rm")

    def myTestContainer = docker.image('node:latest')
    myTestContainer.pull()

    myTestContainer.inside("--link ${mysql.id}:mysql") {
      sh 'npm install --only=dev'
      sh 'npm test'
    }
  }
  stage('docker build/push') {
    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
      def app = docker.build("santiagogustavo/docker-nodejs-demo:${commit_id}", '.').push()
    }
  }
}

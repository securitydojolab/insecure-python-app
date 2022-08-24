pipeline {
    agent  any
    stages {
        stage ("Build Checkout") {
            steps {
                git branch: 'main',
                    url: 'https://github.com/justmorpheus/insecure-python-app.git'

            }
        }
    }
}

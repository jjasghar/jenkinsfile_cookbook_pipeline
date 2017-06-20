node {
    stage('Checkout code') {
        git 'REPO'
    }
}
pipeline {
    agent {
        docker {
            reuseNode false
            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'
            image 'chef/chefdk'
        }
    }
    triggers {
        pollSCM('H * * * *')
    }
    stages {
        stage('\u27A1 Dependencies for Docker and ChefDK') {
            steps {
                sh '''apt-get update
apt-get install -y sudo git build-essential apt-transport-https ca-certificates curl software-properties-common'''
            }
        }
        stage('\u27A1 Install Docker-CE') {
            steps {
                sh '''curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce'''
            }
        }
        stage('\u27A1 Start Docker') {
            steps {
                sh 'service docker start'
            }
        }
        stage('\u27A1 Verify Docker') {
            steps {
                sh 'docker run --rm hello-world'
            }
        }
        stage('\u27A1 Verify ChefDK') {
            steps {
                sh '''/opt/chefdk/embedded/bin/chef --version
/opt/chefdk/embedded/bin/cookstyle --version
/opt/chefdk/embedded/bin/foodcritic --version'''
            }
        }
        stage('\u27A1 Verify Kitchen') {
            steps { sh 'KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen list'
            }
        }
        stage('\u27A1 Run test-kitchen') {
            steps {
                parallel(
                    "centos": {
                        sh '''KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen test centos'''
                    },
                    "debian": {
                        sh '''KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen test debian'''
                    },
                    "fedora": {
                        sh '''KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen test fedora'''
                    },
                    "opensuse": {
                        sh '''KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen test opensuse'''
                    },
                    "ubuntu": {
                        sh '''KITCHEN_LOCAL_YAML=.kitchen.dokken.yml /opt/chefdk/embedded/bin/kitchen test ubuntu'''
                    },
                )
            }
        }
        stage('\u27A1 Upload to Chef Server') {
            steps { sh 'chef exec knife cookbook upload COOKBOOKNAME -o ../'
            }
        }
    }
}

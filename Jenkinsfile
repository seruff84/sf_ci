pipeline {
    agent any
    stages {
        stage('1-Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'seruff_ssh', 
                    url: 'git@github.com:seruff84/sf_ci.git'
                sh "ls -lat"
            }
        }
        stage('2-Docker_bulid') {
            steps {
                sh  'docker build . -t seruff/my_nginx'
                sh 'docker run --name my-app -d -p 9889:80 seruff/my_nginx'
            }
        }
        stage('3-test1') {
	    steps {
		script {
		     def response = httpRequest responseHandle: 'NONE', url: 'http://localhost:9889/index.html', wrapAsMultipart: false
                     if (response.status == 200) {
                        withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
		            sh  ("""
                                     curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='response code 200'
                            """)
                        }
                          return    
		     } else if (response.status != 200) {
                        withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
		            sh  ("""
                                     curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='response code not 200'
                            """)
                         }
                         return    
                       }
		}
	    }
        }

        stage('4-test2') {
            steps {
		script {
                    def md5 = sh (         
                         script: 'curl -s http://localhost:9889/index.html | md5sum | awk \'{print $1 " index.html"}\' | md5sum -c | awk \'{print $2}\'',
                         returnStdout: true
                    ).trim()    
                    println ('Status: '+md5)  
                    if (md5 == 'OK') {

                        withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
		            sh  ("""
                                     curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='MD5 OK'
                            """)
                        }
                        return
                     } else {
                        withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
		            sh  ("""
                                     curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='MD5 not OK'
                            """)
                        }
                        return
                     }

                 }
             }
        }
        stage('5-Clean') {
            steps {
                sh '''            
                    docker container stop my-app
                    docker container rm my-app
                    docker image rm seruff/my_nginx

                '''
            }
        }
   }
    post {
        success { 
            withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
            sh  ("""
                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : OK *Published* = YES'
            """)
        }
    }

        aborted {
            withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
            sh  ("""
                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : `Aborted` *Published* = `Aborted`'
            """)
        }
     
     }
     failure {
        withCredentials([string(credentialsId: 'chatWebid', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
        sh  ("""
           curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC  *Branch*: ${env.GIT_BRANCH} *Build* : `not OK` *Published* = `no`'
        """)
        }
    }  
}
}

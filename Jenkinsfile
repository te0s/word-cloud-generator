pipeline {
    agent any
    stages {
        stage ('Test') {
            agent {
                dockerfile { filename 'dockerfile' 
                            args '--network host'
                           }
            }
            steps {
                sh '''
                make lint && make test
                 if [ $? -ne 0 ];
                    then
                        echo "Test error"
                        exit 1
                    fi
                export GOPATH=$WORKSPACE/go
                export PATH="$PATH:$(go env GOPATH)/bin"               
                go get github.com/tools/godep
                go get github.com/smartystreets/goconvey
                go get github.com/GeertJohan/go.rice/rice  
                go get github.com/wickett/word-cloud-generator/wordyapi
                go get github.com/gorilla/mux
                sed -i "s/1.DEVELOPMENT/1.$BUILD_NUMBER/g" static/version
                GOOS=linux GOARCH=amd64 go build -o ./artifacts/word-cloud-generator -v 
                md5sum artifacts/word-cloud-generator
                ls -l artifacts/
                gzip -f ./artifacts/word-cloud-generator
                cat static/version
                '''
                nexusArtifactUploader artifacts: [[artifactId: 'word-cloud-generator', classifier: '', file: 'artifacts/word-cloud-generator.gz', type: 'gz']], credentialsId: 'uploader', groupId: "$git_branch", nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'word-cloud-build', version: '1.$BUILD_NUMBER'
            }
        }
        stage('Testing') {
            agent {
                dockerfile { filename 'alpine/alpinedockerfile' 
                            args '--network host'
                           }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'downloader', usernameVariable: 'nexus_user', passwordVariable: 'nexus_password')])
                {
                  sh '''
                   curl -u ${nexus_user}:${nexus_password} -X GET "http://localhost:8081/repository/word-cloud-build/$git_branch/word-cloud-generator/1.$BUILD_NUMBER/word-cloud-generator-1.$BUILD_NUMBER.gz" -o /opt/wordcloud/word-cloud-generator.gz
                   if [[ $? -ne 0 ]];
                   then
                       echo "File not found"
                       exit 1
                   else
                       gunzip -f /opt/wordcloud/word-cloud-generator.gz
                       chmod +x /opt/wordcloud/word-cloud-generator
                       /opt/wordcloud/word-cloud-generator &
                       sleep 5
                       res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://localhost:8888/version | jq '. | length'`
                       if [[ "1" != "$res" ]]; then 
                          exit 98
                       fi
                       res=`curl -s -H "Content-Type: application/json" -d '{"text":"ths is a really really really important thing this is"}' http://localhost:8888/api | jq '. | length'`
                       if [[ "7" != "$res" ]]; then
                          exit 99
                       fi
                       sleep 90
                   fi
                   
                  '''
                }
            }
        }
    }
}
node () {
    def workspace = pwd()
    def content = "PROJECT: ${PROJECT_NAME}&BUILD: ${BUILD_NUMBER}&Status: ${BUILD_STATUS}&URL: http://jenkins.royale.com/blue/organizations/jenkins/${PROJECT_NAME}/detail/${PROJECT_NAME}/${BUILD_NUMBER}/pipeline/"

    stage ('checkout') {
        git branch: 'master', credentialsId: '99130ab1-7e3b-4305-8748-a342c064d9a8', url: 'http://gitsrv01.royale.com/front-end/system-oversea-new.git'
    }

    stage ('install') {
        try {
            sh 'npm install'
        } catch(err) {
            sh 'echo "npm install error"'
            def c = "npm install error&"$content
            sh 'bash ansible/notify.sh "$c"'
            throw err
            sh 'exit 1'
        }
    }
    stage ('build') {
        try {
            sh '''
                rm -rf dist
                npm run build
            '''
        } catch(err) {
            sh 'echo "npm run build error"'
            def c = "npm run build error&"$content
            sh 'bash ansible/notify.sh "$c"'
            throw err
            sh 'exit 1'
        }
    }
    stage ('package') {
        try {
            sh '''
                filename=oas-$(date '+%Y%m%d%H%M%S').zip
                cd dist && zip -qr ${filename} *
                cd ..
            '''
        } catch(err) {
            sh 'echo "package error"'
            def c = "package error&"$content
            sh 'bash ansible/notify.sh "$c"'    
            throw err
            sh 'exit 1'
        }
    }
    stage ('update') {
        try {
            sh '''
                src_file=$(ls -rht dist/oas-*.zip | head -n 1)
                dest_file=/data/server_new/${src_file#dist/}
                dt=$(date '+%Y%m%d%H%M%S')
                ansible-playbook -i ansible/hosts ansible/deploy.yml -v --extra-var "src_file=$(pwd)/${src_file} dest_file=${dest_file} arch_file=oas-${dt}.zip project=oas"
                def c = "success&"$content
                sh 'bash ansible/notify.sh "$c"'            
            '''
        } catch(err) {
            sh 'echo "update error"'
            def c = "update error&"$content
            sh 'bash ansible/notify.sh "$c"'              
            throw err
            sh 'exit 1'
        }
    }
}
node("docker") {
    docker.withRegistry('devopsrobo', 'savindra') {
    
        git url: "https://github.com/gangavh/dockerExample.git", credentialsId: 'gangavh'
    
        sh "git rev-parse HEAD > .git/commit-id"
        def commit_id = readFile('.git/commit-id').trim()
        println commit_id
    
        stage "build"
        def app = docker.build "DevOpsRobo"
    
        stage "publish"
        app.push 'master'
        app.push "${commit_id}"
    }
}
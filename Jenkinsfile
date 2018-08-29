@Library('alauda-cicd') _
def IMAGE
def RELEASE_BUILD
def DOCKER_CONTEXT

pipeline {
    
    agent any
    environment {
        DOCKER_CONTEXT = 'docker'
    }
    stages {
        stage('Checkout') {
            
            steps {
                echo 'Checkout'
                //checkout([$class: 'GitSCM', branches: [[name: '*/master']], browser: [$class: 'GithubWeb', repoUrl: 'https://github.com/marsfun/aml-models-repo'], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/marsfun/aml-models-repo']]])
                //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'sharedlib', url: 'https://bitbucket.org/yuanfangalauda/aml-models-repo/']]])
            }
        }    
        stage('busi') {
            steps{
                script{
                    echo 'busi'
                    sh """
                    cat deploy/meta.yaml
                    """

                    def getyamlparam = { String p ->
                        datas = readYaml file: 'deploy/meta.yaml'
                        echo datas

                        echo "metaClass=" datas.get(p)
                        // datas.metaClass.p
                        datas.get(p)
                    }


                    def getparam = { String param ->
                        def matcher = readFile('deploy/meta.xml') =~ "<$param>(.+)</$param>"
                        matcher?matcher[0][1]:null
                    }
                    def sedcmd=""
                    def getCmd = { String old, name ->
                        v = getparam(name)
                        cmd = String.format("sed -i 's/@%s@/%s/g' deploy/modelserver.yaml",name,v)
                        if (old != "") {
                            old + " && " + cmd
                        }else{
                            cmd
                        }
                    }
                    ns1 = getyamlparam('namespace')
                    echo '---'
                    echo ns1

                    sedcmd = getCmd(sedcmd,"namespace")
                    sedcmd = getCmd(sedcmd,"modelname")
                    sedcmd = getCmd(sedcmd,"modelversion")
                    sedcmd = getCmd(sedcmd,"grpcport")
                    sedcmd = getCmd(sedcmd,"restfulport")
                    // print final yaml
                    sh """
                    $sedcmd
                    cat deploy/modelserver.yaml
                    """
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    echo 'Building'
                    def getparam = { String param ->
                        def matcher = readFile('deploy/meta.xml') =~ "<$param>(.+)</$param>"
                        matcher?matcher[0][1]:null
                    }
                    def model = getparam('modelname')
                    def mversion = getparam('modelversion')
                    if (! mversion){
                        mversion = "latest"
                    }
                    IMAGE = deploy.dockerBuild(
                            "docker/Dockerfile", //Dockerfile
                            "docker", // build context
                            "index.alauda.cn/alaudak8s/${model}-modelserver", // repo address
                            mversion, // tag
                            "max-alaudak8s", // credentials for pushing
                        ).setArg("MODEL", model)
                    IMAGE.start().push()
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying'
                    deploy.setupProd()
                    try {
                        sh "kubectl apply -f deploy/modelserver.yaml"
                    } 
                    catch (Exception exc) {
                        echo "error: ${exc}"
                        throw exc
                    }
                }
            }
        }
    }
}
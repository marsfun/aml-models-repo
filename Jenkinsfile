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

                    def getparam = { String param ->
                        def matcher = readFile('deploy/meta.xml') =~ "<$param>(.+)</$param>"
                        matcher?matcher[0][1]:null
                    }


                    def sedcmd=""
                    def getCmd = { String old, k,v ->
                        // v = getparam(name)
                        cmd = String.format("sed -i 's/@%s@/%s/g' deploy/modelserver.yaml",k,v)
                        if (old != "") {
                            old + " && " + cmd
                        }else{
                            cmd
                        }
                    }

                    yamldata = readYaml file: 'deploy/meta.yaml'
                    println('class = %s',yamldata.getClass())
                    echo yamldata.namespace
                    echo yamldata.modelname
                    echo yamldata.modelversion
                    echo yamldata.grpcport+''
                    echo yamldata.restfulport+''
                    echo '---'

                    sedcmd = getCmd(sedcmd,'namespace',yamldata.namespace)
                    sedcmd = getCmd(sedcmd,'modelname',yamldata.modelname)
                    sedcmd = getCmd(sedcmd,'modelversion',yamldata.modelversion)
                    sedcmd = getCmd(sedcmd,'grpcport',yamldata.grpcport+'')
                    sedcmd = getCmd(sedcmd,'restfulport',yamldata.restfulport+'')
                    
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
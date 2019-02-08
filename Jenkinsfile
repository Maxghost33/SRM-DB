def label = "mypod-${UUID.randomUUID().toString()}"


podTemplate(label: label, containers: [
  containerTemplate(name: 'python-alpine', image: 'ghostgoose33/python-alp:v3', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'ghostgoose33/docker-in:v1', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
], serviceAccount: "jenkins") 
{

def dockerRegistry = "100.71.71.71:5000"
def Creds = "git_cred"
def projName = "db-service"
def imageVersion = "latest"
def imageName = "100.71.71.71:5000/db-service:${imageVersion}"
def imageN = '100.71.71.71:5000/db-service:'

properties([
    parameters([
	stringParam(
            defaultValue: '', 
            description: 'TAG_Change', 
            name: 'service')
    ])
])


node(label)
{
    try{
        stage("Pre-Test"){
            dir('db'){
            git(branch: "test", url: 'https://github.com/Kv-045DevOps/SRM-DB.git', credentialsId: "${Creds}")
            imageTagDB = (sh (script: "git rev-parse --short HEAD", returnStdout: true))
            tmp = "1"
	    pathTocodedb = pwd()
            }
        }
        stage("Test image_regisrty_check"){
            container("python-alpine"){
                check_new = (sh (script: "python3 /images-registry-test.py db-service ${imageTagDB}", returnStdout:true).trim())
                echo "${check_new}"
            }
        }
        
        stage ("Unit Tests"){
            sh 'echo "Here will be unit tests"'
        }
        stage("Test code using PyLint and version build"){
			container('python-alpine'){
				pathTocode = pwd()
				sh "python3 /pylint-test.py ${pathTocodedb}/app/routes.py"
			}
        }
        stage("Build docker image"){
			container('docker'){
				pathdocker = pwd()
                                if ("${tmp}" == "${check_new}"){
					container("docker"){
					sh "docker images"
                                	sh "cat /etc/docker/daemon.json"

					sh "docker build ${pathTocodedb} -t ${imageN}${imageTagDB}"
                			sh "docker build ${pathTocodedb}/init-container/ -t ${dockerRegistry}/init-container:${imageTagDB}"
					sh "docker images"
				    
					sh "docker push ${imageN}${imageTagDB}"
                			sh "docker push ${dockerRegistry}/init-container:${imageTagDB}"
					sleep 20
					}
					build(job: 'test_e2e', parameters: [[$class: 'StringParameterValue', name:"imageTagDB_", value: "${imageTagDB}"],
									   [$class: 'StringParameterValue', name:"imageTagUI_", value: "${params.imageTagUI_}"],
									   [$class: 'StringParameterValue', name:"imageTagGET_", value: "${params.imageTagGET_}"],
									   [$class: 'StringParameterValue', name:"service", value: "db"]], wait: true)
        			} else {
            				echo "NO"
        			}
				
			}
        }
    }
    catch(err){
        currentBuild.result = 'Failure'
    }
}
}






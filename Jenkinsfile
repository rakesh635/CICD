properties([pipelineTriggers([githubPush()])])
podTemplate(label: 'mypod1', containers: [
    containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.6.2-jdk-11', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'roffe/kubectl:v1.13.2', command: '', ttyEnabled: true),
    containerTemplate(name: 'tomcat8', image: 'tomcat:8.0', command: '', ttyEnabled: true)
],
volumes: [
	hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
]
)
{
    node('mypod1') {
		
		def app1_name = 'todobackend'
		def app2_name = 'todoui'
		def app1_image_tag = "rakesh635/${app1_name}:v${env.BUILD_NUMBER}"
		def app2_image_tag = "rakesh635/${app2_name}:v${env.BUILD_NUMBER}"
		def app1_dockerfile_name = 'Dockerfile-todobackend'
		def app2_dockerfile_name = 'Dockerfile-todoui'
		def app1_container_name = 'todobackend'
		def app2_container_name = 'todoui'
	
        stage('Application Code Checkout from Git') {
			checkout scm
		
		}
		/*stage('Test with Maven/H2') {
			container('maven'){
				dir ("./${app1_name}") {
					//sh ("mvn dependency::tree")
					sh ("mvn -X test -Dspring.profiles.active=dev")
				}
			}
		}*/
		stage('Test with Maven/PSQL') {
			container('kubectl'){
				withKubeConfig([credentialsId: 'GKEcluster',
				serverUrl: 'https://35.200.147.114',
				contextName: 'qaenv',
				clusterName: 'qaenv',
				namespace: 'qadeploy'
				]){
					sh("kubectl apply -f postgres_test.yml")
				} 		
			}
			container('maven'){ 
				dir ("./${app1_name}") {
					sh ("mvn test -Dspring.profiles.active=prod -Dspring.datasource.url=jdbc:postgresql://${env.PSQL_TEST}/${env.DB_NAME} -Dspring.datasource.username=${env.DB_USERNAME} -Dspring.datasource.password=${env.DB_PASSWORD}")
				}
			}
		}
		stage('Build with Maven') {
			container('maven'){
				dir ("./${app1_name}") {
					
					sh ("mvn -B -DskipTests clean package")
				}
				dir ("./${app2_name}") {
					
					sh ("mvn -B -DskipTests clean package")
				}
			}
		}
		stage('Build Docker Image') {
			container('docker'){
				sh("docker build -f ${app1_dockerfile_name} -t ${app1_image_tag} .")
				sh("docker build -f ${app2_dockerfile_name} -t ${app2_image_tag} .")
			}
		}
		stage('Push Docker Image to Docker Registry') {
			container('docker'){
				docker.withRegistry('', 'dockerlogin') {
                    //sh 'docker push rakesh635/$APPNAME:$BUILD_NUMBER'
                    //sh 'docker push rakesh635/$APPNAME:latest'
					sh("docker push ${app1_image_tag}")
					sh("docker push ${app2_image_tag}")
                }
			}
		}
		stage('Deploy Application on K8s') {
			container('kubectl'){
				withKubeConfig([credentialsId: 'GKEcluster',
				serverUrl: 'https://35.200.147.114',
				contextName: 'qaenv',
				clusterName: 'qaenv',
				namespace: 'qadeploy'
				]){
					sh("kubectl apply -f configmap.yml")
					sh("kubectl apply -f secret.yml")
					sh("kubectl apply -f postgres.yml")
					sh("kubectl apply -f ${app1_name}.yml")
					sh("kubectl set image deployment/${app1_name} ${app1_container_name}=${app1_image_tag}")
					sh("kubectl apply -f ${app2_name}.yml")
					sh("kubectl set image deployment/${app2_name} ${app2_container_name}=${app2_image_tag}")
				}     
			}
        }
    }
}

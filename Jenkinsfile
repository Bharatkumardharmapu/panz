pipeline {
  agent any
  triggers {
    pollSCM '* * * * *'
  }
  stages {
    stage('SonarQube Analysis') {
      steps {
        sh '''
         echo Restore started on `date`.
         dotnet sonarscanner begin /k:"sample" /d:sonar.host.url=$sonar_url /d:sonar.login=$token
         dotnet restore panz.csproj
         dotnet build panz.csproj -c Release
         dotnet sonarscanner end /d:sonar.login=$token 
        
        '''
      }
    }
    stage('Dotnet Publish') {
      steps {
        sh 'dotnet publish panz.csproj -c Release'
      }   
    }
   stage('Docker build and push') {
      steps {
        sh '''
         whoami
         echo $access_key
         aws configure set aws_access_key_id $access_key
         aws configure set aws_secret_access_key $secret_key
         aws configure set default.region ap-south-1
         DOCKER_LOGIN_PASSWORD=$(aws ecr get-login-password  --region ap-south-1)
         docker login -u AWS -p $DOCKER_LOGIN_PASSWORD https://971076122335.dkr.ecr.ap-south-1.amazonaws.com
         docker build -t 971076122335.dkr.ecr.ap-south-1.amazonaws.com/sample:SAMPLE-PROJECT-${BUILD_NUMBER} .
         docker push 971076122335.dkr.ecr.ap-south-1.amazonaws.com/sample:SAMPLE-PROJECT-${BUILD_NUMBER}
          
	  '''
     }   
   }
    stage('ecs deploy') {
      steps {
        sh '''
          chmod +x changebuildnumber.sh
          ./changebuildnumber.sh $BUILD_NUMBER
	  sh -x ecs-auto.sh
          '''
     }    
    }
}
}

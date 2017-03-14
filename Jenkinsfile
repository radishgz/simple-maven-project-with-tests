#!groovy
node {
   def gitUrl
   def svnUrl
   def creid
   def mvnHome
   stage("get env") {
   echo  '1'
   try{
   gitUrl="${GITURL}"
   }catch (exc) {
       gitUrl=null
   }
    echo  '2'
   try{
    svnUrl="${SVNURL}"}
   catch (exc) {
       svnUrl=null
   }
   
    echo  '3'

   try{
     creid="${CREDENTIALS}"
     //  echo  'CREDENTIALS'
     //  echo creid
   }catch (exc) {
       echo "Caught: ${exc}"
        creid=null
   }
      echo  '4'
   try {
       echo MVNNAME
   }catch (exc) {
       error "must define MVNNAEM ,the MAVEN configure name is Jenkins."
   }
  
 
  // echo 'gitUrl' gitUrl 
   //echo 'creid' creid 
   //echo 'gitUrl' gitUrl 
   //echo ''
  
   echo  '-------'
   }
   
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      //git 'http://10.0.2.50:180/xiehq/moskito'
      //git 'https://github.com/radishgz/simple-maven-project-with-tests.git'
      
     // withCredentials([usernamePassword(credentialsId: '8b24228d-e871-4178-82a4-be8b14ca34cf', passwordVariable: 'xhq4y323', usernameVariable: 'xiehq')]) {
    // some block
       echo  '-------'

    if (creid != null){
 
      // withCredentials([usernameColonPassword(credentialsId: creid, variable: '')]) {
         if (gitUrl != null){
            echo  'using credential to git'+gitUrl

            git credentialsId:creid, url:gitUrl
        }
         else if(svnUrl != null)   {
            echo  'using credential to svn'+svnUrl


        checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '',  \
             excludedRevprop: '', excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', \
            locations: [[credentialsId: creid, depthOption: 'infinity', ignoreExternalsOption: true, local: '.', \
            remote: svnUrl]], \
             workspaceUpdater: [$class: 'UpdateUpdater']])	

            // svn svnUrl
        }else{
            error 'Please input svnUrl or gitUrl'

        }
       //}
    }else{
        if (gitUrl != null){
                        echo  'no credential git'+gitUrl

            git gitUrl
        }
         else if(svnUrl != null)   {
                         echo  'no credential svn'+svnUrl

             svn svnUrl
        }else{
            error 'Please input svnUrl or gitUrl'

        }
        
    }    

      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool MVNNAME //'Maven-3.3.9'
      //readMavenPom
    }
 
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
 
   
  stage('Static Check') {
      
 configFileProvider([configFile(fileId: '657387f5-3698-4f0c-a34c-6773568fd612', targetLocation: '../findbugsfilter.xml')]) {
        }
           sh "'${mvnHome}/bin/mvn' -X org.codehaus.mojo:findbugs-maven-plugin:3.0.4:check -Dfindbugs.includeFilterFile=../findbugsfilter.xml -Dfindbugs.xmlOutput=true "
    
  }
stage( "zip"){
  //sh 'rm temp.zip'
  //build.number='${BUILD_NUMBER}'
  //pom = readMavenPom file: 'pom.xml'
  //def filename
  filename=env.BUILD_TAG+".zip"
  echo filename
  echo env.BUILD_TAG
  
  zip archive: true, dir: 'target', glob: '', zipFile: filename
  }
  
  stage("sonar"){
    def   URL
    URL="${SONARHOSTURL}"
    def   PASSWORD
    PASSWORD="${SONARPASSOWRD}"
    def   LOGIN
    LOGIN="${SONARLOGIN}"
    
    withSonarQubeEnv('mySonar') {
// requires SonarQube Scanner for Maven 3.2+
//    sh '${mvnHome}/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
    
    //export sonar.host.url= '10.211.55.44:9000';
    sh "'${mvnHome}/bin/mvn' -X org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar -Dsonar.host.url='${URL}' -Dsonar.login='${LOGIN}' -Dsonar.password='${PASSWORD}'"

}
}

}
  

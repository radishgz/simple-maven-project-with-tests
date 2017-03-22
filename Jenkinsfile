#!groovy

// create by xiehq @2017-03
def usage = {
    echo " the script will checkout source and execute "
    echo " 1. maven's clean and package goals "
    echo " 2. Findbus scan with include filter "
    echo " 3. send content to sonar "
    echo "Environment variables"
    echo " GITURL(option), git url of source code ,GITURL and SVNURL must and only one should be set"
    echo " MVNNAME(required), jenkins's maven install name "
    echo " SVNURL(option), svn url. the url not used when GITURL be set"
    echo " CREDENTIALS(option), the crendential will used to checkout sourcecode , it be defind in Jenkins and should have username and password"
    echo " SONARHOSTURL(), Sonar host url"
    echo " SONARPASSOWRD() Sonar passowrd"
    echo " SONARLOGIN, Sonar login name"
    echo " FINDBUGSFILTER, findbugs include filter xml file id ,the file is a Jenkins config file "

}

node {
    def gitUrl
    def svnUrl
    def creid
    def mvnHome

    stage('Preparation') { // for display purposes
        echo '1'
        try {
            gitUrl = "${GITURL}"
        } catch (exc) {
            gitUrl = null
        }
        try {
            svnUrl = "${SVNURL}"
        }
        catch (exc) {
            svnUrl = null
        }


        try {
            creid = "${CREDENTIALS}"
            //  echo  'CREDENTIALS'
            //  echo creid
        } catch (exc) {
            echo "Caught: ${exc}"
            creid = null
        }
        try {
            echo MVNNAME
        } catch (exc) {
            usage
            error "must define MVNNAEM ,the MAVEN configure name is Jenkins."

        }

        if (creid != null && creid != "") {

            // withCredentials([usernameColonPassword(credentialsId: creid, variable: '')]) {
            if (gitUrl != null && gitUrl != "") {
                echo 'using credential to git' + gitUrl

                git credentialsId: creid, url: gitUrl
            } else if (svnUrl != null && svnUrl != "") {
                echo 'using credential to svn' + svnUrl


                checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '',    \
               excludedRevprop: '', excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '',   \
              locations: [[credentialsId: creid, depthOption: 'infinity', ignoreExternalsOption: true, local: '.',   \
              remote: svnUrl]],   \
               workspaceUpdater: [$class: 'UpdateUpdater']])

                // svn svnUrl
            } else {
                usage
                error 'Please input svnUrl or gitUrl'

            }
            //}
        } else {
            if (gitUrl != null && gitUrl != "") {
                echo 'no credential git' + gitUrl

                git gitUrl
            } else if (svnUrl != null && svnUrl != "") {
                echo 'no credential svn' + svnUrl

                svn svnUrl
            } else {
                usage
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
        //if (isUnix()) {
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
        //} else {
        //    bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
        //}
    }


    stage('Static Check') {
        // sh "curl -L -o ./FingbusFilter.xml '${FINDBUGSFILTER}'"
       

        try {
             configFileProvider([configFile(fileId: FINDBUGSFILTER, targetLocation: './findbugsfilter.xml')]) {
                      sh "'${mvnHome}/bin/mvn'  org.owasp:dependency-check-maven:1.4.5:check -Ddependency-check-format=XML -DreportOutputDirectory=./target"
                      sh "'${mvnHome}/bin/mvn'  org.codehaus.mojo:findbugs-maven-plugin:3.0.4:findbugs -Dfindbugs.includeFilterFile=./findbugsfilter.xml -Dfindbugs.xmlOutput=true -Dfindbugs.pluginList=/root/findsecbugs-plugin-1.5.0.jar"
            
             }
        }
        finally {
            stage("Archive") {
                //sh 'rm temp.zip'
                //build.number='${BUILD_NUMBER}'
                //pom = readMavenPom file: 'pom.xml'
                //def filename
                filename = env.BUILD_TAG + ".zip"
                echo filename
                //echo env.BUILD_TAG
                post {
                     always {
                        Output './*'
                         }
                 }
                //zip archive: true, dir: 'target', glob: '', zipFile: filename
            }
        }

    }

    
    stage("build Image"){
    agent {
    docker {
        label '10.0.2.50'
        sh "docker pull tomcat:"
      }
    }
    }
    
    stage("Sonar") {
        def URL
        URL = "${SONARHOSTURL}"
        def PASSWORD
        PASSWORD = "${SONARPASSOWRD}"
        def LOGIN
        LOGIN = "${SONARLOGIN}"

 // requires SonarQube Scanner for Maven 3.2+
//    sh '${mvnHome}/bin/mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'

            //export sonar.host.url= '10.211.55.44:9000';
            sh "'${mvnHome}/bin/mvn'  org.sonarsource.scanner.maven:sonar-maven-plugin:3.0.2:sonar -Dsonar.host.url='${URL}' -Dsonar.login='${LOGIN}' -Dsonar.password='${PASSWORD}'"

         
    }

}
  

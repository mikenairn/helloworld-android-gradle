/**
* Android Jenkinsfile
*/
node("android"){
  stage("Checkout"){
    checkout scm
  }

  stage ("Prepare"){
    if(params.FH_CONFIG_CONTENT) {
        writeFile file: 'app/src/main/assets/fhconfig.properties', text: params.FH_CONFIG_CONTENT
    }
    sh "tree"
  }

  stage("Build"){
    sh 'chmod +x ./gradlew'
    if (params.BUILD_CONFIG == 'release') {
      sh './gradlew clean assembleRelease' // builds app/build/outputs/apk/app-release.apk file
    } else {
      sh './gradlew clean assembleDebug' // builds app/build/outputs/apk/app-debug.apk
    }
    sh "tree"
  }

  def keyStoreId = params.BUILD_CREDENTIAL_ID
  def keyAlias = params.BUILD_CREDENTIAL_ALIAS ?: ''
  stage("Sign"){
    if (params.BUILD_CONFIG == 'release') {
        signAndroidApks (
            keyStoreId: keyStoreId,
            keyAlias: keyAlias,
            apksToSign: "**/*-unsigned.apk",
            // uncomment the following line to output the signed APK to a separate directory as described above
            // signedApkMapping: [ $class: UnsignedApkBuilderDirMapping ],
            // uncomment the following line to output the signed APK as a sibling of the unsigned APK, as described above, or just omit signedApkMapping
            // you can override these within the script if necessary
            // androidHome: '/usr/local/Cellar/android-sdk'
        )
    } else {
      println('Debug Build - Using default developer signing key')
    }
  }

  stage("Analyze"){
    try {
        analyzeBuild binaryPath: "$WORKSPACE/app/build/outputs/apk/debug/app-debug.apk", platform: 'android'
    } catch(Error e) {
        e.printStackTrace()
    }
  }

 stage("Archive"){
    if (params.BUILD_CONFIG == 'release') {
        archiveArtifacts artifacts: 'app/build/outputs/apk/**/app-release.apk', excludes: 'app/build/outputs/apk/*-unaligned.apk'
    } else {
        archiveArtifacts artifacts: 'app/build/outputs/apk/**/app-debug.apk', excludes: 'app/build/outputs/apk/*-unaligned.apk'
    }
  }
}

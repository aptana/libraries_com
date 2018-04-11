#! groovy

// Keep logs/reports/etc of last 15 builds, only keep build artifacts of last 2 builds
properties([buildDiscarder(logRotator(numToKeepStr: '15', artifactNumToKeepStr: '2'))])

timestamps {
	// Need maven 3.3+
	node('edtftpjpro && linux && eclipse && jdk') {
		stage('Checkout') {
			// checkout scm
			// Hack for JENKINS-37658 - see https://support.cloudbees.com/hc/en-us/articles/226122247-How-to-Customize-Checkout-for-Pipeline-Multibranch
			checkout([
				$class: 'GitSCM',
				branches: scm.branches,
				extensions: scm.extensions + [[$class: 'CleanCheckout'], [$class: 'CloneOption', honorRefspec: true, noTags: true, reference: '', shallow: true, depth: 30, timeout: 30]],
				userRemoteConfigs: scm.userRemoteConfigs
			])
		}

		stage('Dependencies') {
			sh 'cp -f $HOME/edtftpjpro/* $WORKSPACE/bundles/com.aptana.ide.libraries.subscription/'
		}

		stage('Build') {
			withEnv(["PATH+MAVEN=${tool name: 'Maven 3.5.0', type: 'maven'}/bin"]) {
				withCredentials([usernamePassword(credentialsId: 'aca99bee-0f1e-4fc5-a3da-3dfd73f66432', passwordVariable: 'STOREPASS', usernameVariable: 'ALIAS')]) {
					sh "mvn -Djarsigner.keypass=${env.STOREPASS} -Djarsigner.storepass=${env.STOREPASS} -Djarsigner.keystore=${env.KEYSTORE} clean verify"
				} // withCredentials
			}
			dir('releng/com.aptana.ide.libraries.subscription.update/target') {
				// FIXME: To keep backwards compatability with existing build pipeline, I probably need to make the "repository" dir be "dist"
				archiveArtifacts artifacts: "repository/**/*"
				def jarName = sh(returnStdout: true, script: 'ls repository/features/com.aptana.ide.feature.libraries.subscription_*.jar').trim()
				def version = (jarName =~ /.*?_(.+)\.jar/)[0][1]
				currentBuild.displayName = "#${version}-${currentBuild.number}"
			}
		}

		// If not a PR, trigger studio3-core build for same branch
		if (!env.BRANCH_NAME.startsWith('PR-')) {
			build job: "../studio3/${env.BRANCH_NAME}", wait: false
		}
	} // node
} // timestamps

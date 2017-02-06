#! groovy
@Library('pipeline-build') _

timestamps() {
	node('edtftpjpro && linux && ant && eclipse && jdk') {
		try {
			stage('Checkout') {
				checkout scm
			}

			stage('Dependencies') {
				sh 'cp -f $HOME/edtftpjpro/* $WORKSPACE/plugins/com.aptana.ide.libraries.subscription/'
			}

			buildPlugin {
				builder = 'com.aptana.ide.libraries.subscription.build'
			}

			// If not a PR, trigger studio3-core build for same branch
			if (!env.BRANCH_NAME.startsWith('PR-')) {
				build job: "Studio/studio3/${env.BRANCH_NAME}", wait: false
			}
		} catch (e) {
			// if any exception occurs, mark the build as failed
			currentBuild.result = 'FAILURE'
			throw e
		} finally {
			step([$class: 'WsCleanup', notFailBuild: true])
		}
	} // node
} // timestamps

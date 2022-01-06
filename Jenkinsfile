pipeline
{
	agent { node { label 'Jenkins' } }
	stages 
	{
				stage('static-analysis')
				{
					steps
					{
						sh '''
							echo "a thing is happening"

							exit 0;
						'''
					}
					post
					{
						always
						{
							recordIssues qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]], tools: [checkStyle(pattern: '**/checkstyle-result.xml, **/eslint-checkstyle.xml'), pmdParser(pattern: '**/target/pmd.xml')]

							bitbucketNotify("[Complete] Static Code Analysis", "StaticAnalysisPhase", false)
							echo "Static analysis result: ${currentBuild.result}"
						}
					}
				}
	}
	post 
	{
		always 
		{
			script
			{
				currentBuild.description = "Overall Result";  // Plugin progress reporter will use this label to display final result in bitbucket
			}

			echo "Pipeline result: ${currentBuild.result}"
			echo "Pipeline currentResult: ${currentBuild.currentResult}"
		}
		unstable
		{
			clearWorkspace()
		}
	}
}

void clearWorkspace()
{
	script 
	{
		sh 'rm -rf .git server client' // reduce the space overhead of pull requests by clearing out the transient parts of the workspace
	}
}

void bitbucketNotify(String msg, String project, boolean unstableIsSuccess)
{
	currentBuild.result = currentBuild.currentResult // declarative pipeline doesn't set the value of currentBuild.result
	currentBuild.description = msg

	echo "Notifying ${currentBuild.result} for ${env.GIT_COMMIT}"
	notifyBitbucket commitSha1: env.GIT_COMMIT, stashServerBaseUrl: '', disableInprogressNotification: false, considerUnstableAsSuccess: unstableIsSuccess, ignoreUnverifiedSSLPeer: true, includeBuildNumberInKey: false, prependParentProjectKey: false, projectKey: project, credentialsId: ''
}

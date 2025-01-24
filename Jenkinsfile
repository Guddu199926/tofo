pipeline {
    agent any

    environment {
        MAIN_BRANCH = 'dev'
        FEATURE_BRANCH = "${env.BRANCH_NAME}"
        GITHUB_API_URL = 'https://api.github.com'
        REPO = 'Guddu199926/tofo' // Replace with your repo details
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Clone the repository
                    checkout scm
                }
            }
        }

        stage('Check Pull Request Approval') {
            when {
                not { branch 'dev' } // Skip main branch
            }
            steps {
                script {
                    // Get PR details using GitHub API
                    def prResponse = httpRequest(
                        url: "${GITHUB_API_URL}/repos/${REPO}/pulls?head=${REPO}:${FEATURE_BRANCH}",
                        authentication: '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' // Add a personal access token in Jenkins credentials
                    )
                    
                    def prs = readJSON(text: prResponse.content)
                    if (prs.size() == 0) {
                        error("No open PR found for branch ${FEATURE_BRANCH}")
                    }

                    // Check if PR is approved
                    def prNumber = prs[0].number
                    def reviewResponse = httpRequest(
                        url: "${GITHUB_API_URL}/repos/${REPO}/pulls/${prNumber}/reviews",
                        authentication: 'github-token'
                    )

                    def reviews = readJSON(text: reviewResponse.content)
                    def approved = reviews.any { it.state == 'APPROVED' }

                    if (!approved) {
                        error("Pull request #${prNumber} for branch ${FEATURE_BRANCH} is not approved.")
                    }
                    echo "Pull request #${prNumber} is approved."
                }
            }
        }

        stage('Merge to Main') {
            when {
                not { branch 'dev' }
            }
            steps {
                script {
                    // Configure Git user
                    sh '''
                        git config user.name "Guddu199926"
                        git config user.email "mannepalliravi999@gmail.com"
                    '''

                    // Checkout the main branch
                    sh "git checkout ${MAIN_BRANCH}"

                    // Merge the feature branch into the main branch
                    sh "git merge --no-ff ${FEATURE_BRANCH} -m 'Merging ${FEATURE_BRANCH} into ${MAIN_BRANCH}'"

                    // Push the changes back to the repository
                    withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASSWORD')]) {
                        sh "git push https://${GIT_USER}:${GIT_PASSWORD}@<repo-url> ${MAIN_BRANCH}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Feature branch ${FEATURE_BRANCH} successfully merged into ${MAIN_BRANCH}"
        }
        failure {
            echo "Failed to merge feature branch ${FEATURE_BRANCH} into ${MAIN_BRANCH}"
        }
    }
}

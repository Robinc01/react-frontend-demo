pipeline {
    agent any

    // --- NEW: Add this options block for branch and tag discovery/build strategies ---
    options {
        // This configures how Jenkins discovers branches and tags, and what build strategies to apply.
        // It's essential for enabling automatic builds on new tags.
        git.branchSources {
            git.add(new GitBranchSource(
                // IMPORTANT: Replace with your actual GitHub repository HTTPS URL
                remote: 'https://github.com/Robinc01/react-frontend-demo.git',
                
                // Include all branches and tags for discovery
                includes: '*',
                // Define traits for discovery and build strategies
                traits: [
                    // This trait enables the automatic build strategy for new tags
                    git.build.strategy.impl.TagStrategyImpl(),
                    // This trait ensures tags are discovered by the multibranch pipeline
                    git.checkout.impl.TagDiscoveryTrait()
                ]
            ))
        }
    }
    // --- END NEW BLOCK ---

    environment {
        PATH = "/var/lib/jenkins/.nvm/versions/node/v22.2.0/bin:$PATH"
    }

    stages {
        stage('Checkout') {
            steps {
                // 'checkout scm' will use the SCM configuration defined in the branchSources block
                checkout scm
            }
        }

        stage('Detect Release Tag & Env') {
            steps {
                script {
                    // Get tag name manually. env.TAG_NAME is also available for tag builds.
                    // Using git describe --tags --exact-match is robust for annotated tags.
                    env.GIT_TAG_NAME = sh(script: "git describe --tags --exact-match || true", returnStdout: true).trim()

                    def tag = env.GIT_TAG_NAME
                    if (!tag) {
                        error("GIT_TAG_NAME is not set. This pipeline must be triggered by a release tag (e.g., release-dev-20250729-1, release-p1-v1.0.0).")
                    }

                    // Detect environment based on tag prefix
                    if (tag.startsWith('release-dev')) {
                        DEPLOY_ENV = 'dev'
                    } else if (tag.startsWith('release-stg')) {
                        DEPLOY_ENV = 'stg'
                    } else if (tag.startsWith('release-prod')) {
                        DEPLOY_ENV = 'prod'
                    } else {
                        error("Unknown or unsupported release tag prefix: ${tag}. Tags must start with 'release-dev' , 'release-stg' etc.")
                    }

                    echo "Triggered by Git tag: ${tag}"
                    echo "🌐 Target Environment: ${DEPLOY_ENV}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to Environment') {
            steps {
                script {
                    if (DEPLOY_ENV == 'dev') {
                        echo "Deploying to DEV server..."
                        sh "scp -r build/* user@dev-server:/var/www/html/"
                    } else if (DEPLOY_ENV == 'stg') {
                        echo "Deploying to STAGING server..."
                        sh "scp -r build/* user@stg-server:/var/www/html/"
                    } else if (DEPLOY_ENV == 'prod') {
                        echo "Deploying to PRODUCTION server..."
                        sh "scp -r build/* user@prod-server:/var/www/html/"
                    }
                }
            }
        }

        stage('Archive Build') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }
    }
}

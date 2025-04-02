pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0431bf5e-5008-40b5-be82-408babc78afb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        NODE_ENV = 'production'  // กำหนด environment default เป็น production
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔧 Checking required files..."
                sh '''
                    test -f index.html || (echo "❌ Missing index.html" && exit 1)
                    test -f netlify/functions/quote.js || (echo "❌ Missing quote function" && exit 1)
                    echo "✅ Build check passed."
                '''
                echo "📦 Installing dependencies..."
                sh '''
                    npm install --production || (echo "❌ npm install failed" && exit 1)
                    echo "✅ Dependencies installed."
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🧪 Running tests..."
                sh '''
                    npm test || (echo "❌ Tests failed" && exit 1)
                    echo "✅ All tests passed."
                '''
            }
        }

        stage('Security Check') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔐 Running security check..."
                sh '''
                    npm audit --production || (echo "❌ Security vulnerabilities found" && exit 1)
                    echo "✅ No security vulnerabilities found."
                '''
            }
        }

        stage('Deploy to Netlify') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🚀 Deploying to Netlify..."
                sh '''
                    npm install netlify-cli || (echo "❌ netlify-cli installation failed" && exit 1)
                    node_modules/.bin/netlify deploy \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --dir=. \
                      --prod || (echo "❌ Deploy failed" && exit 1)
                    echo "✅ Deployment successful."
                '''
            }
        }

        stage('Post Deploy') {
            steps {
                echo "✅ Deployment complete! Your app is live."
                echo "📦 Archiving build artifacts..."
                archiveArtifacts artifacts: '**/dist/*.zip', allowEmptyArchive: true
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline finished successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}

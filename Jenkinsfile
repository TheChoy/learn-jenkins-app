pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = credentials('netlify-site-id')  // ใช้ Jenkins credentials สำหรับ site ID
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')  // ใช้ Jenkins credentials สำหรับ Auth Token
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
                sh 'npm install --production'  // ติดตั้งเฉพาะ dependencies สำหรับ production
                echo "✅ Dependencies installed."
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
                    npm install netlify-cli
                    node_modules/.bin/netlify deploy \
                      --auth=$NETLIFY_AUTH_TOKEN \
                      --site=$NETLIFY_SITE_ID \
                      --dir=. \
                      --prod
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
            mail to: 'developer@example.com', subject: 'Build Success', body: 'The build and deploy were successful!'
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
            mail to: 'developer@example.com', subject: 'Build Failed', body: 'The build failed. Please check the logs.'
        }
    }
}

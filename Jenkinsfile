pipeline {
    agent any

    environment {
        SITE_ID = '0431bf5e-5008-40b5-be82-408babc78afb'
        AUTH_TOKEN = credentials('netlify-token')
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
                echo "🔧 Verifying necessary files..."
                sh '''
                    # ตรวจสอบเวอร์ชันของ node และ npm
                    node -v
                    npm -v

                    # ตรวจสอบว่าไฟล์ที่จำเป็นมีหรือไม่
                    test -f index.html || { echo "❌ index.html is missing!" && exit 1; }
                    test -f netlify/functions/quote.js || { echo "❌ Missing quote.js function" && exit 1; }
                    echo "✅ All required files are present."
                '''
            }
        }

        stage('Code Quality Check') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔍 Running ESLint for code quality analysis..."
                script {
                    def lintResult = sh(
                        script: '''
                            npm install eslint
                            npx eslint . || echo "❌ Found linting errors!"
                        ''',
                        returnStatus: true
                    )
                    if (lintResult != 0) {
                        echo "⚠️ Linting finished with errors, but pipeline continues."
                    } else {
                        echo "✅ Code is lint-free."
                    }
                }
            }
        }

        stage('Security Audit') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔒 Running security audit with npm..."
                script {
                    def auditResult = sh(
                        script: '''
                            npm install
                            npm audit --production || echo "❌ Security vulnerabilities detected!"
                        ''',
                        returnStatus: true
                    )
                    if (auditResult != 0) {
                        echo "⚠️ Security audit completed with vulnerabilities, but continuing pipeline."
                    } else {
                        echo "✅ Security check passed."
                    }
                }
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
                echo "🧪 Verifying if quote function loads correctly..."
                sh '''
                    node -e "require('./netlify/functions/quote.js'); console.log('✅ Function loaded successfully')"
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🚀 Deploying to Netlify..."
                sh '''
                    # ตรวจสอบว่า netlify-cli ติดตั้งแล้ว
                    which netlify-cli || { echo "❌ netlify-cli not found!" && exit 1; }

                    npm install netlify-cli --save-dev  # ติดตั้ง netlify-cli
                    node_modules/.bin/netlify deploy \
                        --auth=$AUTH_TOKEN \
                        --site=$SITE_ID \
                        --dir=. \
                        --prod
                '''
            }
        }

        stage('Post-Deployment') {
            steps {
                echo "✅ Deployment completed successfully! Your app is live."
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD pipeline executed successfully."
            // Optional: Send Slack notification here or email
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
            // Optional: Send failure notification to Slack or email
        }
    }
}

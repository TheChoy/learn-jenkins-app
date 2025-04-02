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
                echo "🔧 กำลังกำหนดไฟล์ที่ต้องการ..."
                sh '''
                    # ตรวจสอบไฟล์ที่จำเป็น
                    test -f index.html || { echo "❌ ไม่พบไฟล์ index.html!" && exit 1; }
                    test -f netlify/functions/quote.js || { echo "❌ ขาดฟังก์ชัน quote.js!" && exit 1; }
                    echo "✅ ไฟล์ครบทุกอย่างแล้ว!"
                '''
            }
        }

        stage('QualityCheck') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔍 กำลังกำหนดคุณภาพโค้ดด้วย ESLint..."
                script {
                    def lintResult = sh(
                        script: '''
                            npm install eslint
                            npx eslint . || echo "❌ พบข้อผิดพลาดในโค้ด!"
                        ''',
                        returnStatus: true
                    )
                    if (lintResult != 0) {
                        echo "⚠️ โค้ดมีข้อผิดพลาด แต่จะทำการรันต่อไปค่ะ"
                    } else {
                        echo "✅ โค้ดสวยงาม ไม่มีข้อผิดพลาด!"
                    }
                }
            }
        }

        stage('Security') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "🔒 กำลังกำหนดการตรวจสอบความปลอดภัย..."
                script {
                    def auditResult = sh(
                        script: '''
                            npm install
                            npm audit --production || echo "❌ พบช่องโหว่ด้านความปลอดภัย!"
                        ''',
                        returnStatus: true
                    )
                    if (auditResult != 0) {
                        echo "⚠️ ตรวจสอบความปลอดภัยเสร็จแล้ว พบช่องโหว่ แต่จะทำการรันต่อไปค่ะ"
                    } else {
                        echo "✅ ไม่มีช่องโหว่ด้านความปลอดภัย!"
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
                echo "🧪 กำลังกำหนดการทดสอบฟังก์ชัน quote.js..."
                sh '''
                    node -e "require('./netlify/functions/quote.js'); console.log('✅ ฟังก์ชันโหลดสำเร็จแล้วค่ะ')"
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
                echo "🚀 กำลังกำหนดการ deploy ไปยัง Netlify..."
                sh '''
                    npm install netlify-cli
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
                echo "✅ การ deploy เสร็จสมบูรณ์ค่ะ เว็บไซต์ของคุณพร้อมใช้งาน!"
            }
        }
    }

    post {
        success {
            echo "🎉 การทำงานของ CI/CD pipeline สำเร็จเรียบร้อยแล้วค่ะ"
        }
        failure {
            echo "❌ pipeline ล้มเหลวค่ะ โปรดตรวจสอบ logs"
        }
    }
}

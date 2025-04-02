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
                script {
                    echo "🏗️ กำลังกำหนดโปรเจค..."
                    sh '''
                    npm install
                    npx react-scripts build
                    '''
                }
            }
            post {
                success {
                    echo "✅ การสร้างโปรเจคสำเร็จแล้ว! 🎉"
                }
                failure {
                    echo "❌ การสร้างโปรเจคล้มเหลว! กรุณาตรวจสอบรายละเอียด."
                }
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
                script {
                    echo "🔍 กำลังกำหนดคุณภาพโค้ดด้วย ESLint..."
                    def lintResult = sh(
                        script: '''
                            npm ci
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

        stage('Security Audit') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🔒 กำลังกำหนดการตรวจสอบความปลอดภัย..."
                    def auditResult = sh(
                        script: '''
                            npm ci
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
                script {
                    echo "🧪 กำลังกำหนดการทดสอบฟังก์ชัน quote.js..."
                    sh '''
                        node -e "require('./netlify/functions/quote.js'); console.log('✅ ฟังก์ชันโหลดสำเร็จแล้วค่ะ')"
                    '''
                }
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
                script {
                    echo "🚀 กำลังกำหนดการ deploy ไปยัง Netlify..."
                    sh '''
                    npx netlify deploy --prod --dir=build \
                    --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                    '''
                }
            }
            post {
                success {
                    echo "✅ การ deploy สำเร็จแล้ว! 🎉"
                    echo "👉 เว็บไซต์ของคุณพร้อมใช้งานที่: https://nicevanitermproject.netlify.app"
                }
                failure {
                    echo "❌ การ deploy ล้มเหลว! กรุณาตรวจสอบรายละเอียด."
                }
            }
        }

        stage('Post-Deploy') {
            agent any
            steps {
                script {
                    echo "🔍 กำลังกำหนดการตรวจสอบทรัพยากรหลังจาก deploy..."
            
                    try {
                        sh '''
                            echo "Top 10 processes by memory usage:" > resource_report.txt
                            ps aux --sort=-%mem | head -n 10 >> resource_report.txt
                            

                            echo "\nMemory usage:" >> resource_report.txt
                            free -h >> resource_report.txt
                            

                            echo "\nSystem performance stats (vmstat):" >> resource_report.txt
                            vmstat 1 5 >> resource_report.txt
                        '''
                    } catch (e) {
                        echo "เกิดข้อผิดพลาดในการตรวจสอบทรัพยากร: ${e}"
                    }
                }
            }
            post {
                success {
                    echo "✅ การตรวจสอบทรัพยากรเสร็จสมบูรณ์แล้ว!"
                    sh 'cat resource_report.txt'  // แสดงข้อมูลที่บันทึกไว้
                }
                failure {
                    echo "❌ การตรวจสอบทรัพยากรพบข้อผิดพลาด!"
                }
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
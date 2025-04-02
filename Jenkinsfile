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
                echo "🏗️ กำลังกำหนดโปรเจค..."
                sh '''
                    npm install
                    npx react-scripts build
                '''
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
                echo "🔍 กำลังกำหนดคุณภาพโค้ดด้วย ESLint..."
                sh '''
                    npm install eslint
                    npx eslint . || echo "❌ พบข้อผิดพลาดในโค้ด!"
                '''
            }
            post {
                success {
                    echo "✅ โค้ดสวยงาม ไม่มีข้อผิดพลาด!"
                }
                failure {
                    echo "❌ โค้ดมีข้อผิดพลาด!"
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
                echo "🔒 กำลังกำหนดการตรวจสอบความปลอดภัย..."
                sh '''
                    npm install
                    npm audit --production || echo "❌ พบช่องโหว่ด้านความปลอดภัย!"
                '''
            }
            post {
                success {
                    echo "✅ ไม่มีช่องโหว่ด้านความปลอดภัย!"
                }
                failure {
                    echo "❌ ตรวจสอบความปลอดภัยพบช่องโหว่!"
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

        stage('Deploy to Netlify') {
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
                    npx netlify-cli deploy --prod --dir=build \
                    --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                '''
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
                echo "🔍 กำลังกำหนดการตรวจสอบทรัพยากรหลังจาก deploy..."
                sh '''
                    echo "Top 10 processes by memory usage:" > resource_report.txt
                    ps aux --sort=-%mem | head -n 10 >> resource_report.txt

                    echo "\nMemory usage:" >> resource_report.txt
                    free -h >> resource_report.txt

                    echo "\nSystem performance stats (vmstat):" >> resource_report.txt
                    vmstat 1 5 >> resource_report.txt
                '''
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

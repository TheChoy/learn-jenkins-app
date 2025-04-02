pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0431bf5e-5008-40b5-be82-408babc78afb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Prepare') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🔧 กำลังเตรียมโปรเจค..."
                    sh '''
                    # ติดตั้ง dependencies ที่จำเป็น
                    npm ci
                    '''
                }
            }
            post {
                success {
                    echo "✅ การเตรียมโปรเจคเสร็จสมบูรณ์"
                }
                failure {
                    echo "❌ การเตรียมโปรเจคล้มเหลว! กรุณาตรวจสอบ logs"
                }
            }
        }

        stage('Build Project') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🏗️ กำลังสร้างโปรเจค..."
                    sh '''
                    # สร้างโปรเจค React
                    npx react-scripts build
                    '''
                }
            }
            post {
                success {
                    echo "✅ การสร้างโปรเจคสำเร็จ! 🎉"
                }
                failure {
                    echo "❌ การสร้างโปรเจคล้มเหลว! กรุณาตรวจสอบ logs"
                }
            }
        }

        stage('Run Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    echo "🧪 กำลังทดสอบโปรเจค..."
                    sh '''
                    # ทดสอบโปรเจค
                    npm test
                    '''
                }
            }
            post {
                success {
                    echo "✅ การทดสอบสำเร็จ! 🎉"
                }
                failure {
                    echo "❌ การทดสอบล้มเหลว! กรุณาตรวจสอบ logs"
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
                    echo "🚀 กำลัง deploy ไปที่ Netlify..."
                    sh '''
                    # ใช้ Netlify CLI ในการ deploy
                    npx netlify deploy --prod --dir=build \
                    --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                    '''
                }
            }
            post {
                success {
                    echo "✅ การ deploy สำเร็จ! 🎉"
                    echo "👉 เว็บไซต์ของคุณพร้อมใช้งานที่: https://nicevanitermproject.netlify.app"
                }
                failure {
                    echo "❌ การ deploy ล้มเหลว! กรุณาตรวจสอบ logs"
                }
            }
        }

        stage('Monitor Server Resources') {
            agent any
            steps {
                script {
                    echo "🔍 กำลังตรวจสอบทรัพยากรเซิร์ฟเวอร์หลังจากการ deploy..."
                    try {
                        sh '''
                        # ตรวจสอบการใช้ทรัพยากรของระบบ
                        echo "Top 10 processes by memory usage:" > resource_report.txt
                        ps aux --sort=-%mem | head -n 10 >> resource_report.txt
                        
                        echo "\nMemory usage:" >> resource_report.txt
                        free -h >> resource_report.txt
                        
                        echo "\nSystem performance stats (vmstat):" >> resource_report.txt
                        vmstat 1 5 >> resource_report.txt
                        '''
                    } catch (e) {
                        echo "❌ เกิดข้อผิดพลาดในการตรวจสอบทรัพยากร: ${e}"
                    }
                }
            }
            post {
                success {
                    echo "✅ การตรวจสอบทรัพยากรสำเร็จ! นี่คือละเอียด:"
                    sh 'cat resource_report.txt'
                }
                failure {
                    echo "❌ ตรวจสอบทรัพยากรเกิดข้อผิดพลาด!"
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline สำเร็จเรียบร้อย! ระบบพร้อมใช้งาน"
        }
        failure {
            echo "❌ Pipeline ล้มเหลว! โปรดตรวจสอบ logs สำหรับรายละเอียด"
        }
    }
}

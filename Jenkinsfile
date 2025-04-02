pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-auth-token')
        NETLIFY_SITE_ID = credentials('netlify-site-id')
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
                    set -x  # เปิดการแสดง log แบบละเอียด
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
                    set -x  # เปิดการแสดง log แบบละเอียด
                    npm install netlify-cli  # ติดตั้ง netlify-cli
                    which netlify-cli || { echo "❌ netlify-cli ไม่พบ!" && exit 1; }  # ตรวจสอบว่า netlify-cli ติดตั้งสำเร็จหรือไม่
                    
                    # ใช้ npx เพื่อเรียกคำสั่ง netlify-cli
                    npx netlify-cli deploy --prod --dir=build \
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
                            set -x  # เปิดการแสดง log แบบละเอียด
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

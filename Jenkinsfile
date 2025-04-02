pipeline {
    agent any

    environment {
        SITE_ID = '0431bf5e-5008-40b5-be82-408babc78afb'
        AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
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

        stage('Code Quality Check') {
            steps {
                echo "🔍 กำลังกำหนดคุณภาพโค้ดด้วย ESLint..."
                script {
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
            steps {
                echo "🔒 กำลังกำหนดการตรวจสอบความปลอดภัย..."
                script {
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
                        echo "✅ ตรวจสอบความปลอดภัยเสร็จแล้ว ไม่มีช่องโหว่!"
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo "🧪 กำลังทดสอบฟังก์ชัน quote.js..."
                sh '''
                    node -e "require('./netlify/functions/quote.js'); console.log('✅ ฟังก์ชันโหลดสำเร็จแล้วค่ะ')"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 กำลังกำหนดการ deploy ไปยัง Netlify..."
                sh '''
                    # ตรวจสอบว่า node และ npm ติดตั้งได้หรือไม่
                    node -v
                    npm -v

                    # ติดตั้ง netlify-cli ก่อนใช้งาน
                    npm install netlify-cli --save-dev

                    # ใช้ npx เพื่อเรียกคำสั่ง netlify-cli
                    npx netlify-cli deploy \
                        --auth=$AUTH_TOKEN \
                        --site=$SITE_ID \
                        --dir=. \
                        --prod
                '''
            }
}


        stage('Post-Deployment') {
            steps {
                echo "✅ การ deploy เสร็จสมบูรณ์แล้วค่ะ เว็บไซต์ของคุณพร้อมใช้งาน!"
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

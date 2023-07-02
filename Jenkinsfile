
pipeline {
    agent none
    stages{
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Manual Approval') {
            steps {
                script {
                    def approval = input(
                        id: 'approval',
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [
                            [$class: 'ChoiceParameterDefinition', 
                             choices: 'Proceed\nAbort', 
                             description: 'Pilih tindakan', 
                             name: 'decision']
                        ]
                    )

                    if (approval == 'Abort') {
                        error('Pipeline dihentikan berdasarkan pilihan pengguna.')
                    }
                }
            }
        }
        stage('Deploy') { 
            agent any
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
        stage('Pause for 1 Minute') {
            steps {
                // Menggunakan langkah sleep untuk menjeda eksekusi selama 1 menit
                sleep time: 1, unit: 'MINUTES'
                // Atau menggunakan langkah timeout dari plugin Build Timeout
                timeout(time: 1, unit: 'MINUTES') {
                    // Tidak perlu melakukan apa-apa di sini, cukup menunggu waktu timeout berakhir
                }
            }
        }
        stage('End Application') {
            steps {
                // Langkah untuk mengakhiri aplikasi secara otomatis setelah 1 menit
                // Contoh: stop server atau layanan yang menjalankan aplikasi
                // sh 'npm stop' atau perintah lainnya sesuai dengan kebutuhan Anda
            }
        }
    }
}
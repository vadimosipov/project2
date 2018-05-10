import java.util.StringJoiner
import java.nio.file.Files
import java.nio.file.Paths
import java.time.LocalDate

pipeline {
    agent any
    triggers { cron('0 15 1 * *') }
    environment {
        LOCAL_STORAGE_DIR = "cells"
    }
    stages {
        stage('Create dir') {
            steps {
                sh "mkdir ${LOCAL_STORAGE_DIR}"
            }
        }
        stage('Download') {
            environment {
                VIGO_CREDENTIALS = credentials('vigo_credentials')
                VIGO_USER = "${env.VIGO_CREDENTIALS_USR}"
                VIGO_PASS = "${env.VIGO_CREDENTIALS_PSW}"
            }
            steps {
                script {
                    def year = LocalDate.now().getYear() - 1
                    def weeks = [15]
                    for (week in weeks) {
                        def startDownloading = { url, body, file -> 
                            sh "curl -X POST ${url} -d \'${body}\' -o ${file}"
                            checkFileSize(file)
                        }
                        generateRequsts(year, week, startDownloading)
                    }
                }
            }
        }
        stage('Save to hdfs') {
            steps {
                echo '1'
                // sh "hdfs dfs -put ${file} /tmp/vosipov"
            }
        }
    }
    post {
        always {
            sendNotification("Your build completed, pipeline: ${currentBuild.fullDisplayName}, please check: ${env.BUILD_URL}") 
        }
    }
}

def generateRequsts(year, week, download) {
    @NonCPS 
    def join = { parameters -> 
        def joiner = new StringJoiner(",", "{", "}");
        for (record in parameters) {
            joiner.add("\"${record.key}\": \"${record.value}\"");
        }                  
        return joiner.toString()
    }

    def url = "uxgeo.vigo.ru/export-data"
    def tasks = readFile("resources/tasks.txt")
    for (taskId in tasks.split("\n")) {
        def params = [
            task_id: "${taskId}",
            mode: "${LOCAL_STORAGE_DIR}", 
            periods: ["${year}-${week}"],
            show_summary: false, 
            scale: 1, 
            format: "csv", 
            client: "${VIGO_USER}",
            key: "${VIGO_PASS}"
        ]
        def body = join(params)
        def file = "${storageDir}/${params.task_id}-${params.periods[0]}.${params.format}"
        download(url, body, file)
    }
}

def checkFileSize(file) {
    def absoluteFilePath = pwd() + "/${file}"
    def size = Files.size(Paths.get(absoluteFilePath))
    if (size < 1024) {
        def warning = "File size is less than 1024 bytes, name: ${currentBuild.fullDisplayName}, please check: ${env.BUILD_URL}"
        sendNotification(warning)
    }
}

def sendNotification(message) {
    try {
        mail to: 'osipov.vad@gmail.com', subject: "Vigo Pipeline", body: message
    } catch(e) {
        echo message
        echo e
    }
}
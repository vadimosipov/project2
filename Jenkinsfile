import java.util.StringJoiner
import java.nio.file.Files
//import java.nio.file.Path
import java.nio.file.Paths

pipeline {
    agent any
    triggers { cron('0 15 1 * *') }
    stages {
        stage('Download') {
            environment {
                VIGO_CREDENTIALS = credentials('vigo_credentials')
                VIGO_USER = "${env.VIGO_CREDENTIALS_USR}"
                VIGO_PASS = "${env.VIGO_CREDENTIALS_PSW}"
            }
            steps {
                script {

                    @NonCPS 
                    def join = { parameters -> 
                        def joiner = new StringJoiner(",", "{", "}");
                        for (record in parameters) {
                            joiner.add("\"${record.key}\": \"${record.value}\"");
                        }                  
                        return joiner.toString()
                    }

                    // def path2 = Paths.get("resources", "tasks.txt") 
                    // def path = new File("resources/tasks.txt").toPath()
                    // println path.toAbsolutePath()
                    // println path2.toAbsolutePath()
                    // def strings = Files.readAllLines(path2)
                    def content = readFile("resources/tasks.txt")
                    echo "${content.getClass()}"
                    echo "${content}"
                    for (taskId in content.split("\n")) {
                        def params = [
                            task_id: "${taskId}",
                            mode: "cells", 
                            periods: ["2017-15"],
                            show_summary: false, 
                            scale: 1, 
                            format: "csv", 
                            client: "${VIGO_USER}",
                            key: "${VIGO_PASS}"
                        ]
                        def data = join(params)
                        def file = "${params.task_id}-${params.periods[0]}.${params.format}"
                        def url = "uxgeo.vigo.ru/export-data"
                        sh "curl -X POST ${url} -d \'${data}\' -o ${file}"
                    }
                /*
                    strings.forEach { taskId -> 
                        def params = [
                            task_id: "${taskId}",
                            mode: "cells", 
                            periods: ["2017-15"],
                            show_summary: false, 
                            scale: 1, 
                            format: "csv", 
                            client: "${VIGO_USER}",
                            key: "${VIGO_PASS}"
                        ]
                        def data = join(params)
                        def file = "${params.task_id}-${params.periods[0]}.${params.format}"
                        def url = "uxgeo.vigo.ru/export-data"
                        sh "curl -X POST ${url} -d \'${data}\' -o ${file}"
                    }
                */
                }
            }
        }
        stage('Save') {
            steps {
                echo '1'
                // sh "hdfs dfs -put ${file} /tmp/vosipov"
            }
        }
    }
    post {
        always {
            mail to: 'osipov.vad@gmail.com',
                 subject: "Completed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Your build completed, please check: ${env.BUILD_URL}"
        }
    }
}
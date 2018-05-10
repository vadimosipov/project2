pipeline {
    agent any
    triggers { cron('0 15 1 * *') }
    stages {
        stage('Checkout') {
            steps {
                git url: ''
            }
        }
        stage('Download') {
            steps {
                script {
                    import java.util.StringJoiner
                    
                    def url = "uxgeo.vigo.ru/export-data"
                    def params = [
                        task_id: "1504108731946657771",
                        mode: "cells", 
                        periods: ["2017-15"],
                        show_summary: false, 
                        scale: 1, 
                        format: "csv", 
                        client: "megafon",
                        key: "mr0keKuZZefWjTux"
                    ]
                    def joiner = new StringJoiner(",", "{", "}");
                    for (record in params) {
                        joiner.add("\"${record.key}\": \"${record.value}\"");
                    }                  
                    def data = joiner.toString()
                    def file = "${params.task_id}-${params.periods[0]}.${params.format}"
                    sh "curl -X POST ${url} -d \'${data}\' -o ${file}"
                    sh "hdfs dfs -put ${file} /tmp/vosipov"
                }
            }
        }
    }
    post {
        
    }
}
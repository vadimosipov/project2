import java.util.StringJoiner

pipeline {
    agent any
    triggers { cron('0 15 1 * *') }
    stages {
        stage('Download') {
            environment {
                VIGO_PASSWORD = credentials('vigo_credentials')
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

                    Path path = new File("resources/tasks.txt").toPath();
                    System.out.println(path.toAbsolutePath());
                    List<String> strings = Files.readAllLines(path);
                    strings.forEach { taskId -> 
                        def params = [
                            task_id: "${taskId}",
                            mode: "cells", 
                            periods: ["2017-15"],
                            show_summary: false, 
                            scale: 1, 
                            format: "csv", 
                            client: "megafon",
                            key: "${VIGO_PASSWORD}"
                        ]
                        def data = join(params)
                        def file = "${params.task_id}-${params.periods[0]}.${params.format}"
                        def url = "uxgeo.vigo.ru/export-data"
                        sh "curl -X POST ${url} -d \'${data}\' -o ${file}"
                    }

                    echo "${VIGO_PASSWORD}"

                    
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
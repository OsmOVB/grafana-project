pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        FLASK_APP_URL = 'http://localhost:5000/alunos'
        GRAFANA_PORT = '3000'
    }

    stages {
        stage('Free Port 3000') {
            steps {
                script {
                    echo "=== Garantindo que a porta 3000 está livre ==="
                    
                    sh '''
                    # Verifica e encerra qualquer processo que esteja usando a porta 3000
                    PORT_IN_USE=$(lsof -t -i:${GRAFANA_PORT})

                    if [ ! -z "$PORT_IN_USE" ]; then
                        echo "A porta ${GRAFANA_PORT} está em uso. Encerrando o processo correspondente..."
                        kill -9 $PORT_IN_USE || echo "Falha ao encerrar processo na porta ${GRAFANA_PORT}."
                        echo "Processo encerrado."
                    else
                        echo "A porta ${GRAFANA_PORT} está livre."
                    fi

                    # Derruba qualquer container Docker que possa estar usando a porta 3000
                    CONTAINER_ID=$(docker ps -q --filter "publish=${GRAFANA_PORT}")
                    if [ ! -z "$CONTAINER_ID" ]; then
                        echo "Container usando a porta ${GRAFANA_PORT} encontrado. Encerrando container..."
                        docker stop $CONTAINER_ID && docker rm $CONTAINER_ID
                        echo "Container encerrado e removido."
                    else
                        echo "Nenhum container em execução na porta ${GRAFANA_PORT}."
                    fi

                    # Derruba todos os containers Docker para evitar conflitos
                    docker-compose down || echo "Nenhum container para remover."
                    '''
                }
            }
        }
        stage('Checkout') {
            steps {
                echo "=== Iniciando o checkout do repositório ==="
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo "=== Construindo os containers ==="
                sh 'docker-compose build'
            }
        }
        stage('Run Containers') {
            steps {
                echo "=== Iniciando os containers ==="
                sh 'docker-compose up -d'
            }
        }
        stage('Run Tests') {
            steps {
                echo "=== Executando os testes ==="
                
                // Aguarda até que o Flask esteja disponível
                sh '''
                for i in {1..10}; do
                    curl --silent --fail ${FLASK_APP_URL} && break || echo "Aguardando o Flask estar disponível..."
                    sleep 5
                done
                '''

                // Executa o arquivo de teste
                sh '''
                python3 -m pip install --no-cache-dir requests
                python3 test_app.py
                '''
            }
        }
    }

    post {
        always {
            echo "=== Derrubando todos os containers ==="
            sh 'docker-compose down'
        }
        success {
            echo "Pipeline executado com sucesso!"
        }
        failure {
            echo "Falha no pipeline. Verifique os logs."
        }
    }
}

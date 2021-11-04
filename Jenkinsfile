pipeline {
    agent none
    environment {
        scenario_id = ''
        execute_id = ''
        GREMLIN_API_KEY = credentials('gremlin-api-key')
        GREMLIN_TEAM_ID = credentials('gremlin-team-id')
    }
    parameters {
        string(name: 'TargetDeployment', defaultValue: 'ratings-v1', description: 'Deployment Name')
		string(name: 'AttackType', defaultValue: 'shutdown', description: 'Chaos Attack Type')        
        string(name: 'Percentage', defaultValue: '100', description: 'Percent of pods to impact')
        string(name: 'Delay', defaultValue: '1', description: 'The number of minutes to delay before shutting down')		
    }
    stages {
        stage('Creating Chaos Shutdown Scenarios') {
            agent any
            steps {
                script {
                    scenario_id = sh (
                        script: "curl -s 'https://api.gremlin.com/v1/scenarios?teamId=${GREMLIN_TEAM_ID}' -H 'Content-Type: application/json;charset=utf-8' -H 'Authorization: Key ${GREMLIN_API_KEY}' -d '{\"name\":\"$AttackType-$TargetDeployment\",\"recommended_scenario_id\":\"\",\"graph\":{\"nodes\":{\"0\":{\"target_definition\":{\"target_type\":\"Kubernetes\" ,\"strategy_type\":\"Random\" ,\"strategy\":{\"attrs\":{} ,\"type\":\"RandomPercent\" ,\"percentage\":\"$Percentage\"} ,\"k8s_objects\":[{\"cluster_id\":\"eks-chaos-aws\" ,\"uid\":\"956f7174-729c-4ed7-aa81-ae2f65f5fee2\" ,\"namespace\":\"default\" ,\"name\":\"$TargetDeployment\" ,\"kind\":\"DEPLOYMENT\" ,\"labels\":{} ,\"annotations\":{} ,\"available_containers\":[] ,\"target_type\":\"Kubernetes\"}] ,\"containerSelection\":{\"selectionType\":\"ALL\" ,\"containerNames\":[]}} ,\"impact_definition\":{\"infra_command_type\":\"$AttackType\" ,\"infra_command_args\":{\"cli_args\":[\"$AttackType\" ,\"-d\" ,\"$Delay\" ,\"-r\"] ,\"type\":\"$AttackType\"}} ,\"type\":\"InfraAttack\" ,\"id\":\"0\"}} ,\"start_id\":\"0\"}}' --compressed",
                        returnStdout: true
                        ).trim()
                    echo "View scenario details at https://app.gremlin.com/scenarios/detail/${scenario_id}"
                }
            }
        }
        stage('Execute Scenario') {
            agent any
            input {
                    message 'Do you want to run the scenario?'
                    parameters {
                        choice(choices: ['yes' , 'no'], name: 'ExecuteScenario', description: '')
                    }
            }
            steps {
                script {
                    if (env.ExecuteScenario=='yes') {
                        build job:'Devops3-Downstream_Scenario_Execution_Job' , 
                        parameters: [ 
                            [ $class: 'StringParameterValue', name: 'Scenario_ID', value: "${scenario_id}" ]
                        ]
                    }
                }
            }
        }
    }
}

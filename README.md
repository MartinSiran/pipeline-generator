# pipeline-generator
Pipeline generator - bachelor thesis

# Usage

1. Head to [demo petriflow engine](https://demo.netgrif.com/) (registration needed)
2. Left menu -> `Processes` -> Upload file button -> upload both files (`pipeline_generator.xml` & `new_step.xml`)
3. Left menu -> `All Cases` -> Plus button -> choose `Pipeline Generator` in newest version -> `Next` -> `Create`
4. Choose system for CI/CD pipeline generation
5. In build, test deploy steps
    - add new steps, write CMD commands (e.g. `mvn test`)
6. Download generated file
    - Note for Jenkins - update agent (e.g. `docker`)

# Processes

## Pipeline Generator
- main process (parent)
- file `pipeline_generator.xml`
- places
    - start
    - chosen system
    - build steps done
    - test steps done
    - config file done
- transitions
    - choose system
        - select system to generate pipeline for
    - build steps
        - add build steps to pipeline
    - test steps
        - add test steps to pipeline
    - deploy steps
        - add deploy steps to pipeline
    - get file
        - download config file for pipeline
- data variables:
    - system
        - Text
        - system to generate pipeline for
        - default: `Jenkins`
    - config_file
        - File
        - generated file to download in last form
    - taskRef0
        - Task Ref
        - used for adding more steps
    - button0
        - Button
        - add new step function
- cache
    - `resultMap`
        - [Groovy map](https://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Map.html})
        - example:
        ```
        [
            build: ["step1", "step2"],
            test: ["step1", "step2"],
            deploy: ["step1"]
        ]
        ```

## New Step
- sub process (child)
- file `new_step.xml`
- transitions
    - t1
        - input command
        - delete step (self)
- data variables:
    - text0
        - Text
        - command for step
    - button0
        - Button
        - delete step (self)
    - parent_id
        - Text
        - parent case id
        - used for deletion of step
    - parent_transition_id
        - Text
        - parent transition id
        - used for deletion of step

# Example

1. Fork and clone [example-project-java](https://github.com/MartinSiran/example-java-project)
2. Clone [jenkins-docker](https://github.com/MartinSiran/jenkins-docker)
3. In `jenkins-docker` directory root
```console
$ docker compose up
$ # alternatively docker-compose up
```
- [if you don't have docker installed](https://www.docker.com/get-started/)
4. Go to http://localhost:8080/ & copy paste `secret-key` from console
5. In Jenkins, install required plugins and create user
6. In Jenkins, click `New Item` on the left, add name
7. In Jenkins, select `Pipeline` and confirm
8. In Jenkins, section `Pipeline`, field `Definition` - change value to `Pipeline script from SCM`
9. In Jenkins, section `Pipeline`, field `SCM` - change value to `Git`
10. In Jenkins, section `Pipeline`, field `Repository URL` - change value to forked repository, e.g. `https://github.com/your-username/example-java-project`
11. In Jenkins, section `Pipeline`, section `Branches to build`, add 2 items, values `*/test-success`, `*/test-error`
12. In Jenkins, save configuration
13. Head to [demo petriflow engine](https://demo.netgrif.com/) (register or sign in)
14. In demo petriflow engine, left menu -> `Processes` -> Upload file button -> upload both `.xml` files
15. In demo petriflow engine, left menu -> `All Cases` -> Add new case (plus button) -> choose `Pipeline Generator` and create
16. In demo petriflow engine, choose system `Jenkins` and fill Jenkins agent field with `docker { image 'maven:3.8.4-openjdk-11-slim' }`
17. In demo petriflow engine, `assign` task `build steps` and `Add step`, fill in value `mvn clean compile`
18. In demo petriflow engine, `assign` task `test steps` and `Add step`, fill in value `mvn test`
19. In demo petriflow engine, `assign` task `deploy steps` and `Add step`, fill in value `mvn compile exec:java -Dexec.mainClass="org.example.App"`
20. In demo petriflow engine, `assign` task `get file`, download `Jenkinsfile`
21. final Jenkinsfile should look like
```
pipeline {
    agent { docker { image 'maven:3.8.4-openjdk-11-slim' } }
    stages {
        stage('build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('deploy') {
            steps {
                sh 'mvn compile exec:java -Dexec.mainClass="org.example.App"'
            }
        }
    }
}
```
22. Add updated `Jenkinsfile` to forked repository `example-java-project` to branches `test-success` & `test-error`
23. In Jenkins, open configured project from dashboard, click `Build Now` twice
24. In Jenkins, one pipeline passed, second failed

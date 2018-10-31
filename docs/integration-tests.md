# Karate Integration Tests

Our API testing framework is [Karate](https://en.wikipedia.org/wiki/Karate_(software)), which is an web-API automation framework created by Intuit, and is based on [Cucumber](https://en.wikipedia.org/wiki/Cucumber_(software)) and Java. As such this will be a Maven project.

## Creating a test project
1. Open a command prompt/terminal
1. Ensure there is an integration-tests folder under your repo root or create it if it does not exist. (As discussed in [Creating a new API: YOUR Project Structure](creating-api.md#YOUR-Project-Structure) )
1. Change into the integration-tests folder
1. Run the command below, replacing *\<SHORT_NAME\>* with the project's short name as defined in YOUR Project Structure:
    ```
    mvn archetype:generate \
    -DarchetypeGroupId=com.intuit.karate \
    -DarchetypeArtifactId=karate-archetype \
    -DarchetypeVersion=0.8.0 \
    -DgroupId=gov.nci.ocpl.api.<SHORT_NAME> \
    -DartifactId=<SHORT_NAME>-integration-tests
    ```
1. ...Now Draw the Rest of the Owl


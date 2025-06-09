// pipeline {
//     agent any

//     environment {
//         BRANCH_NAME = "${env.BRANCH_NAME}"
//         IS_PR = false
//     }

//     stages {
//         stage('Determine Event Type') {
//             steps {
//                 script {
//                     // Determine if this is a PR build (for multibranch, use env.CHANGE_BRANCH)
//                     if (env.CHANGE_BRANCH) {
//                         IS_PR = true
//                         BRANCH_NAME = env.CHANGE_BRANCH
//                         echo "📦 Pull Request from branch: ${BRANCH_NAME}"
//                     } else if (env.BRANCH_NAME) {
//                         echo "🔁 Branch build: ${env.BRANCH_NAME}"
//                     } else {
//                         error("Unsupported event: Could not detect branch from environment")
//                     }

//                     // Allowed branches to build
//                     def allowedBranches = ['main', 'branch1', 'branch2']
//                     if (!allowedBranches.contains(BRANCH_NAME)) {
//                         echo "⚠️ Skipping branch ${BRANCH_NAME} as it is not in the allowed list."
//                         currentBuild.result = 'SUCCESS'
//                         error("Branch ${BRANCH_NAME} is not allowed, skipping the build.")
//                     }
//                 }
//             }
//         }

//         stage('Checkout Code') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Set up Python & Run Tests') {
//             steps {
//                 sh '''
//                     set -e
//                     python3 -m venv venv
//                     . venv/bin/activate
//                     python --version
//                     pip install --upgrade pip
//                     pip install pytest
//                     pip list
//                     pytest tests/test_calculator_logic.py
//                 '''
//             }
//         }
//     }

//     post {
//         success {
//             echo "✅ Pipeline completed successfully for branch: ${BRANCH_NAME}"
//         }
//         failure {
//             echo "❌ Pipeline failed for branch: ${BRANCH_NAME}"
//         }
//     }
// }



pipeline {
    agent any

    environment {
        // Define environment variables. BRANCH_NAME will be dynamically set.
        BRANCH_NAME = "${env.BRANCH_NAME}"
        IS_PR = false // Flag to determine if the build is for a Pull Request
    }

    stages {
        stage('Determine Event Type') {
            steps {
                script {
                    // Start a try-catch block to gracefully handle issues in this stage
                    try {
                        // Determine if this is a PR build (for multibranch, use env.CHANGE_BRANCH)
                        if (env.CHANGE_BRANCH) {
                            IS_PR = true
                            BRANCH_NAME = env.CHANGE_BRANCH
                            echo "📦 Pull Request from branch: ${BRANCH_NAME}"
                        } else if (env.BRANCH_NAME) {
                            echo "🔁 Branch build: ${env.BRANCH_NAME}"
                        } else {
                            // If no branch name can be detected, it's an unsupported event
                            currentBuild.result = 'FAILURE' // Explicitly mark build as failure
                            error("Unsupported event: Could not detect branch from environment.")
                        }

                        // Allowed branches to build
                        def allowedBranches = ['main', 'branch1', 'branch2']
                        // Check if the current branch is in the list of allowed branches
                        if (!allowedBranches.contains(BRANCH_NAME)) {
                            echo "⚠️ Skipping branch ${BRANCH_NAME} as it is not in the allowed list."
                            currentBuild.result = 'FAILURE' // Explicitly mark build as failure
                            error("Branch '${BRANCH_NAME}' is not allowed, skipping the build.")
                        }
                    } catch (e) {
                        // Catch any unexpected errors in this stage
                        currentBuild.result = 'FAILURE'
                        currentBuild.description = "Failed in 'Determine Event Type' stage: ${e.message}"
                        error("Error in 'Determine Event Type' stage: ${e.message}")
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    // Start a try-catch block for the checkout process
                    try {
                        echo "Checking out source code..."
                        checkout scm // Performs the SCM checkout
                        echo "Code checkout successful."
                    } catch (e) {
                        // If checkout fails, mark the build as failed and provide a detailed message
                        currentBuild.result = 'FAILURE'
                        currentBuild.description = "Failed to checkout code: ${e.message}"
                        error("Checkout Code stage failed: ${e.message}")
                    }
                }
            }
        }

        stage('Set up Python & Run Tests') {
            steps {
                script {
                    // Wrap the shell commands in a script block with try-catch for granular error reporting
                    try {
                        echo "Setting up Python environment and running tests..."
                        sh '''
                            # Exit immediately if a command exits with a non-zero status.
                            set -e

                            echo "Creating Python virtual environment..."
                            python3 -m venv venv || { echo "Failed to create virtual environment"; exit 1; }

                            echo "Activating virtual environment..."
                            . venv/bin/activate || { echo "Failed to activate virtual environment"; exit 1; }

                            echo "Python version:"
                            python --version || { echo "Failed to get Python version"; exit 1; }

                            echo "Upgrading pip..."
                            pip install --upgrade pip || { echo "Failed to upgrade pip"; exit 1; }

                            echo "Installing pytest..."
                            pip install pytest || { echo "Failed to install pytest"; exit 1; }

                            echo "Installed Python packages:"
                            pip list || { echo "Failed to list pip packages"; exit 1; }

                            echo "Running tests with pytest..."
                            pytest tests/test_calculator_logic.py || { echo "Pytest tests failed!"; exit 1; }

                            echo "Python setup and tests completed successfully."
                        '''
                    } catch (e) {
                        // If any command in the 'sh' block fails, this catch block will execute
                        // Set the overall build result to FAILURE
                        currentBuild.result = 'FAILURE'
                        // Provide a more specific description about where the failure occurred
                        currentBuild.description = "Python setup or test stage failed: ${e.message}"
                        // Use 'error()' to fail the pipeline and print a clear message to the log
                        error("Set up Python & Run Tests stage failed. Check console output for details.")
                    }
                }
            }
        }
    }

    post {
        // Actions to perform after the entire pipeline has completed
        success {
            // Runs if all stages completed successfully
            echo "✅ Pipeline completed successfully for branch: ${BRANCH_NAME}"
            // Optionally, you can add more actions like sending success notifications
        }
        failure {
            // Runs if any stage failed
            echo "❌ Pipeline failed for branch: ${BRANCH_NAME}. See logs for details."
            // You can add more actions here, e.g.,
            // mail to: 'your-email@example.com',
            // subject: "Jenkins Pipeline FAILED: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
            // body: "Pipeline for branch ${BRANCH_NAME} failed. Check ${env.BUILD_URL}"
        }
        unstable {
            // Runs if the build is 'unstable' (e.g., tests passed but some other checks failed)
            echo "🟡 Pipeline completed with unstable status for branch: ${BRANCH_NAME}"
        }
        aborted {
            // Runs if the build was manually aborted
            echo "🛑 Pipeline aborted for branch: ${BRANCH_NAME}"
        }
    }
}

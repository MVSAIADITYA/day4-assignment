Steps to Set Up CI Pipeline Using CircleCI on GCP
1. Create a GCP Project
Make sure you have a Google Cloud project with the necessary services (Compute Engine, Cloud Storage, etc.).
Enable billing, and ensure you have permissions to manage resources.
2. Set Up Google Compute Engine (GCE) Instance
Create a VM instance on Google Compute Engine.
Ensure SSH access is enabled for the instance.
Install any required software on the instance, such as Docker or specific runtimes for your application.
3. CircleCI Setup
Sign up or log in to CircleCI.
Connect your version control system (e.g., GitHub, Bitbucket) to CircleCI.
In your repository, create a .circleci directory, and inside it, create a config.yml file for CircleCI configuration.
4. Define Pipeline Stages in .circleci/config.yml
Example config.yml:
yaml
Copy
version: 2.1

executors:
  docker-executor:
    docker:
      - image: 'circleci/python:3.9'  # Specify the Docker image for your application
    working_directory: ~/repo

jobs:
  build:
    executor: docker-executor
    steps:
      - checkout  # Get the code from the repository
      - run:
          name: Install dependencies
          command: |
            pip install -r requirements.txt  # Install app dependencies
      - run:
          name: Build Docker Image
          command: |
            docker build -t gcr.io/YOUR_PROJECT_ID/your-app .  # Build the Docker image
    
  test:
    executor: docker-executor
    steps:
      - checkout
      - run:
          name: Install dependencies
          command: |
            pip install -r requirements.txt  # Install app dependencies
      - run:
          name: Run tests
          command: |
            pytest  # Run tests using pytest or your test framework
    
  deploy:
    docker:
      - image: 'google/cloud-sdk:alpine'  # Use the Google Cloud SDK image for deployment
    steps:
      - checkout
      - run:
          name: Authenticate with GCP
          command: |
            echo $GCLOUD_SERVICE_KEY | base64 --decode > /tmp/gcloud-key.json
            gcloud auth activate-service-account --key-file /tmp/gcloud-key.json
            gcloud config set project YOUR_PROJECT_ID
      - run:
          name: Deploy to GCP Compute Engine
          command: |
            gcloud compute ssh YOUR_INSTANCE_NAME --zone YOUR_ZONE --command "docker pull gcr.io/YOUR_PROJECT_ID/your-app && docker run -d gcr.io/YOUR_PROJECT_ID/your-app"
          
workflows:
  version: 2
  build_deploy:
    jobs:
      - build
      - test:
          requires:
            - build
      - deploy:
          requires:
            - test
5. Configure Build Triggers
CircleCI automatically triggers a build for each push to the repository. For more advanced triggers (e.g., only when certain branches or tags are updated), use the filters configuration:

yaml
Copy
workflows:
  version: 2
  build_deploy:
    jobs:
      - build:
          filters:
            branches:
              only:
                - main  # Trigger builds only for the main branch
      - test:
          requires:
            - build
          filters:
            branches:
              only:
                - main  # Trigger tests only for the main branch
      - deploy:
          requires:
            - test
          filters:
            branches:
              only:
                - main  # Trigger deployment only for the main branch
6. Commit Changes with Clear Message
Commit the changes you made to the .circleci/config.yml file (or any other configuration files):

bash
Copy
git add .circleci/config.yml
git commit -m "Setup CircleCI pipeline with build, test, and deploy stages"
7. Push Changes to Your Repository
Push the changes to your feature branch:

bash
Copy
git push origin feature-gcp-ci
8. Create a Pull Request to Merge feature-gcp-ci into main
After pushing your feature branch, go to your Git repository (e.g., GitHub) and create a pull request (PR) to merge feature-gcp-ci into the main branch. This will trigger CircleCI to run the pipeline and deploy to Google Compute Engine after successful tests.god
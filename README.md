Apache Airflow is an open source software that allows developers to build data pipelines by writing Python scripts. These scripts, called directed acyclic graphs or DAGs, tell the Airflow engine a list of tasks to execute, the order in which to execute the tasks, and a schedule of how often each should run. Cloud Composer is a managed Airflow service from Google Cloud Platform (GCP) which runs on Kubernetes.
CloudBuild - automatically test and deploy Airflow data pipelines to Composer
    Defining a DAG
    Testing a DAG
    Deploying DAGs to Composer
    Using Cloud Build to automate the testing and deployment process

1. Defining a DAG
The DAG we’ll be deploying takes advantage of
PostgresToGoogleCloudStorageOperator. It applies SQL to the source data set stored under the table name loan_data. In our example below, the operator transfers the entire dataset to Cloud Storage along with adding a computed field.

2. Testing a DAG
To enable Cloud Build to automatically test our DAG above, we’ll add a single validation test to our repository. Borrowing from this Medium post, this test will use DagBag to import DAGs, checking to make sure our DAG doesn’t contain a cycle.

3. Deploying a DAG to composer.
Cloud Composer is a managed Airflow service that runs on Kubernetes. Each Composer environment config holds reference to a Cloud Storage directory where it will retrieve DAGs from. This source DAG directory is created upon initialization of a Composer Environment. We can find the exact path to this directory by looking up the Configuration in Composer Environments -> Environment Configuration.

Any DAGs that we drop in the directory above will automatically be picked up and brought into our Composer environment.

Our above DAG is located in a ‘/dags’ directory within a local git repository, and we can deploy it to Composer using gsutil to sync our local repository to Cloud Storage. 

Gsutil and rsync are syncing our local DAG repo to Cloud Storage. Composer automatically recognizes changes to the Cloud Storage repo, bringing in new or updated DAGs. Instead of deploying from our local computer, we can take this same command and turn it into an automated deploy stage using Cloud Build.

The test step uses pip to install our dependencies and pytest to run the test suite. Because our trigger occurs on push to any branch, the test step will run every time.
The deploy stage uses gsutil to copy DAGs from the git repository to our source destination. Our trigger is set to occur on push to any branch. However for some steps (such as deploy), we only want them to run on certain branches. Cloud Build only allows branches to be specified within the Cloud Build trigger, rather than within the `.yaml` file. Our wish (but is not currently supported by Cloud Build) would be to specify the branch in our deploy step:

- name: gcr.io/google.com/cloudsdktool/cloud-sdk id: Deploy branch: master
Two workarounds for the above limitation:

For a single repository, add multiple triggers to Cloud Build. You could have one trigger for all branches, a separate trigger for development branch, and a third trigger for master branch. Each trigger references its own `.yaml` configuration file, and contains custom steps for that specific branch.
Use a bash conditional determined by the branch name to determine whether to complete the step.

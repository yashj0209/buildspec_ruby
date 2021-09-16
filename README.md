# Getting Started with OCI DevOps
This is a sample project, using Ruby with Sinatra framework to create a simple Hello World web application. With [OCI DevOps Service](https://www.oracle.com/devops/devops-service/) and this project, you'll be able to build this application and store the executable in [Oracle Artifact Registry](https://docs.oracle.com/en-us/iaas/artifacts/using/overview.htm).

In this example, you'll build a simple Ruby web application, test it locally and push your built artifact to the OCI Artifact Registry using the OCI DevOps service.

## Running the example locally 

### Clone the Repository 
The first step is to download the repository to your local workspace.

```
git clone git@github.com:yashj0209/buildspec_ruby.git
cd buildspec_ruby
```

### Install the Requirements and Run the App (TO EDIT)
Open a terminal and test out the Ruby Hello World web app example.

***STEPS Here****

Now, confirm you can access the web app running in the browser


And open your browser to http://localhost:4567/ 


## Build and test the app in OCI DevOps

Now that you've seen how you can locally test this app, let's build our CI/CD pipeline in OCI DevOps Service.

### Create External Connection to your Git repository 

1. Create a [DevOps Project](https://docs.oracle.com/en-us/iaas/Content/devops/using/devops_projects.htm) or use and an existing project. 
2. Create an External Connection to your Github repository in your DevOps project.
   - Create a Personal Access Token (PAT): https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token
   - In the OCI Console, Go to Identity & Security -> Vault and create a [Vault]( https://docs.oracle.com/en-us/iaas/Content/KeyManagement/Concepts/keyoverview.htm) in compartment of your own choice.
   - Create a Master Key that will be used to encrypt the PATs. 
   - Select Secrets from under Resources and create a secret using PAT obtained from Github account.
   - Make a note of the OCID of the secret.
   - Now, go to the desired project and select External Connection from the resources.
   - Select type as Github and provide OCID of the secret under Personal Access Token.
   - Finally, allow Build Service (dynamic group with DevOps Resources) to use PAT secret by writing a policy in the root compartment as: ``` Allow dynamic-group dg-with-devops-resources to manage secret-family in tenancy```

### Setup your Build Pipeline

Create a new Build Pipeline to build, test and deliver artifacts. 

#### Managed Build stage

In your Build Pipeline, first add a Managed Build stage. 

1. The Build Spec File Path is the relative location in your repo of the build_spec.yml . Leave the default, for this example. 
2. For the Primary Code Repository 
   - Select connection type as Github
   - Select external connection you created above
   - Give the repo URL to the repo which contains build_spec.yml file.
   - Select main branch.

#### Create an Artifact Registry (TO EDIT)

Create an [Artifact Registry](https://docs.oracle.com/en-us/iaas/Content/Registry/Tasks/registrycreatingarepository.htm) for the ruby-helloworld-example container image built in the Managed Build stage.
1. You can name the repo: ```python-flask-example```. The path to the repo will be REGION/TENANCY-NAMESPACE/python-flask-example
2. To pull the container image without authorization, set the repository access to public. Under "Actions", choose ```Change to public```.

#### Create a DevOps Artifact for your container image repository (TO EDIT)

The version of the container image that will be delivered to the OCI repository is defined by a parameter in the Artifact URI that matches a Build Spec File exported variable (the variable ```version``` in this example) or Build Pipeline parameter name.

In the project, under Artifacts, create a DevOps Artifact to point to the Container Registry repository location you just created above. Enter the information for the Artifact location:

1. Name: python-flask-example container
2. Type: Container image repository
3. Path: REGION/TENANCY-NAMESPACE/python-flask-example
4. Replace parameters: Yes, substitute placeholders

#### Add a Deliver Artifacts stage (TO EDIT)

Let's add a Deliver Artifacts stage to your Build Pipeline to deliver the ```python-flask-example``` container to an OCI repository.

The Deliver Artifacts stage maps the output Artifacts from the Managed Build stage with the version to deliver to a DevOps Artifact resource, and then to the OCI repository (OCIR).

Add a Deliver Artifacts stage to your Build Pipeline after the Managed Build stage. To configure this stage:

1. In your Deliver Artifacts stage, choose ```Select Artifact```
2. From the list of artifacts select the ```python-flask-example container``` artifact that you created above
3. Assign the container image outputArtifact from the ```build_spec.yml``` to the DevOps project artifact. For the "Build config/result Artifact name" enter: ```flask_python``` (This name should be the same as the one mentioned in the outputArtifact section of the build_spec.yml file).

### Run your Build in OCI DevOps

#### From your Build Pipeline, choose Manual Run

Use the Manual Run button to start a Build Run

Manual Run will use the Primary Code Repository, will start the Build Pipeline, first running the Managed Build stage, followed by the Deliver Artifacts stage.

After the Build Pipeline execution is complete, we can view the file stored in the OCI Artifact Registry, which can then be downloaded to local workspace (Under Ellipses , choose ``` Download```).







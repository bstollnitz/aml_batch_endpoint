# How to deploy using a batch endpoint and MLflow

This project shows how to deploy a Fashion MNIST MLflow model using a batch endpoint. Endpoint 1 demonstrates the simplest scenario, and endpoint 2 demonstrates how to wrap the deployment with custom code.

## Blog post

To learn more about the code in this repo, check out the accompanying blog post: https://bea.stollnitz.com/blog/aml-batch-endpoint/

## Setup

- You need to have an Azure subscription. You can get a [free subscription](https://azure.microsoft.com/en-us/free) to try it out.
- Create a [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).
- Create a new machine learning workspace by following the "Create the workspace" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/quickstart-create-resources). Keep in mind that you'll be creating a "machine learning workspace" Azure resource, not a "workspace" Azure resource, which is entirely different!
- Install the Azure CLI by following the instructions in the [documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
- Install the ML extension to the Azure CLI by following the "Installation" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli).
- Install and activate the conda environment by executing the following commands:

```
conda env create -f environment.yml
conda activate aml_batch_endpoint
```

- Within VS Code, go to the Command Palette clicking "Ctrl + Shift + P," type "Python: Select Interpreter," and select the environment that matches the name of this project.
- In a terminal window, log in to Azure by executing `az login --use-device-code`.
- Set your default subscription by executing `az account set -s "<YOUR_SUBSCRIPTION_NAME_OR_ID>"`. You can verify your default subscription by executing `az account show`, or by looking at `~/.azure/azureProfile.json`.
- Set your default resource group and workspace by executing `az configure --defaults group="<YOUR_RESOURCE_GROUP>" workspace="<YOUR_WORKSPACE>"`. You can verify your defaults by executing `az configure --list-defaults` or by looking at `~/.azure/config`.
- You can now open the [Azure Machine Learning studio](https://ml.azure.com/), where you'll be able to see and manage all the machine learning resources we'll be creating.
- Install the [Azure Machine Learning extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai), and log in to it by clicking on "Azure" in the left-hand menu, and then clicking on "Sign in to Azure."

# Training and inference on your development machine

## Endpoint 1

- Under "Run and Debug" on VS Code's left navigation, choose the "Train endpoint 1 locally" run configuration and press F5. A `model` folder is created with the trained model.
- You can analyze the metrics logged in the `mlruns` directory with the following command:

```
mlflow ui
```

- Make a local prediction by running the "Test endpoint 1 locally" configuration.

## Endpoint 2

- Run the "Train endpoint 2 locally" configuration. Two new folders are created, `pyfunc_model` and `pytorch_model`. The `pyfunc_model` is the outer one that we'll use to make predictions.
- You can analyze the metrics logged in the `mlruns` directory with the following command:

```
mlflow ui
```

- Make a local prediction with the following command:

```
cd aml_batch_endpoint/endpoint_2
mlflow models predict --model-uri pyfunc_model --input-path "../test_data/images.csv" --content-type csv --env-manager local
```

# Deploying in the cloud using Azure ML

Create the compute cluster.

```
cd ..
az ml compute create -f cloud/cluster-cpu.yml
```

## Endpoint 1

```
cd endpoint_1
```

Create the model resource on Azure ML.

```
az ml model create --path model/ --name model-batch-1 --version 1 --type mlflow_model
```

Create the endpoint.

```
az ml batch-endpoint create -f cloud/endpoint.yml
az ml batch-deployment create -f cloud/deployment.yml --set-default
```

Invoke the endpoint.

```
az ml batch-endpoint invoke --name endpoint-batch-1 --input ../test_data/images
```

Here's how you delete the endpoint when you're done:

```
az ml batch-endpoint delete --name endpoint-batch-1 -y
```

## Endpoint 2

```
cd ../endpoint_2
```

Create the model resource on Azure ML.

```
az ml model create --path pyfunc_model/ --name model-batch-2 --version 1 --type mlflow_model
```

Create the endpoint.

```
az ml batch-endpoint create -f cloud/endpoint.yml
az ml batch-deployment create -f cloud/deployment.yml --set-default
```

Invoke the endpoint.

```
az ml batch-endpoint invoke --name endpoint-batch-2 --input ../test_data/images.csv --input-type uri_file
```

Invoke the endpoint using a curl command.

```
az ml data create -f cloud/data-invoke-batch.yml
chmod +x invoke.sh
./invoke.sh
```

Here's how you delete the endpoint when you're done:

```
az ml batch-endpoint delete --name endpoint-batch-2 -y
```

## Related resourcse

- [Azure ML endpoints](https://docs.microsoft.com/en-us/azure/machine-learning/concept-endpoints?WT.mc_id=aiml-44164-bstollnitz)
- [Deploy MLflow models](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-deploy-mlflow-models?tabs=fromjob%2Cmir%2Ccli?WT.mc_id=aiml-44164-bstollnitz)

# How To Load Project From Cluster A To Cluster B

#### prerequisite - 
* Git - 
  * You need to upload all the files that include in the project YAML on Cluster A to Git ot other remote storage.
  * Add to the function YAML attributes, with_repo = True - If the function is not a single file function, and it requires access to multiple files/libraries in the project.
* Remote Storage - 
  * All artifacts need to be upload to remote storage ot locally saved on Cluster B
* load project or create one, Example:
This important becuse you need to create an project object instance (MlrunProject object)
````
project = mlrun.get_or_create_project(name='nyc-taxi-gilad',context='./nyc-gilad')
````
# Process on Cluster A 
For this Example i allready run a project that called nyc-taxi and he had two functions (taxi: mlrun job, model-serving: mlrun serving) and one trained model.
## Create Project YAML -
On thie paragraph you would explain how to save your project YAML, and what are the option that you have.

### Set Function - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=set_function#mlrun.projects.MlrunProject.set_function)
Save fucntions objects in the project YAML, there are three options to set function or by fucntion file, or by function YAML or by fucntion object
* **Parameters  -**
  * func – function object or spec/code url, None refers to current Notebook
  * name – name of the function (under the project)
  * kind – runtime kind e.g. job, nuclio, spark, dask, mpijob default: job
  * image – docker image to be used, can also be specified in the function object/yaml
  * handler – default function handler to invoke (can only be set with .py/.ipynb files)
  * with_repo – add (clone) the current repo to the build source
  * requirements – list of python packages or pip requirements file path

1. **Set function file  -**
Need to define all relavnt function parameters, for the func parameter define the function python file as you can see in the examlpe below.
 
````
project.set_function("gen_breast_cancer.py", "gen-breast-cancer", image="mlrun/mlrun")# set function file gen_breast_cancer.py, named gen_breast_cancer with mlrun/mlrun image
project.set_function("trainer.py", "trainer", 
                     handler="train", image="mlrun/mlrun")
project.set_function("serving.py", "serving", image="mlrun/mlrun", kind="serving")

````
* If you want to deploy a serving function you must add a model before deployment using the [add model method](https://docs.mlrun.org/en/latest/api/mlrun.runtimes.html?highlight=add_model#mlrun.runtimes.ServingRuntime.add_model) to the function object, for generate function object you can simple use this example:
````
project.get_function('model-serving')
````

2. **Set function YAML -**
Need to define the func parameter with the function YAML file  and name, as you can see in the examlpe below (other attributes can be define and will ovrite the exciting values).
````
project.set_function(func='taxi.yaml',name='taxi')
````
* To make the function Yaml you would need to use the [export method](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=export#mlrun.projects.MlrunProject.export) this method export function object to YAML file, Example:
````
project.get_function('model-serving').export(target='./nyc-gilad/serving.yaml')
````
* You need add manuly the model_file for **serving function only**, Example:
````
project.spec.artifacts[0]['spec']['model_file']='model.pkl'
````

3. **Set function object -**
Need to define the func parameter with the function object as you can see in the examlpe below.
````
project.set_function(func=taxi_object)
````
* it convert the object to dictionary and saved it to the project YAML file, save fucntion YAML in to the project YAML
* You need add manuly the model_file for **serving function only**, Example:
````
project.spec.artifacts[0]['spec']['model_file']='model.pkl'
````
### Set Artifact - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=set_artifact#mlrun.projects.MlrunProject.set_artifact)
Save artifacts objects in the project YAML, there are two options to set artifcats or by artifcat file, or by artifcat object.
**important note -** You need to store your artifacts on a remote storage or locally on your target cluster (cluster B) 
* **Parameters  -**
  * key – artifact key/name
  * artifact – mlrun Artifact object (or its subclasses)
  * target_path – absolute target path url (point to the artifact content location)
  * tag – artifact tag
1. **Set Artifact file --** 
Need to define all relavnt artifacts parameters, except artifact parameter that relevant only when you set artifact by its object, Example:
````
project.set_artifact(key='training_model', target_path='<target path>')
````
* This option all the artifacts load as artifacts objects not and model or dataset

2. **Set Artifact object --** 
Need to define artifact object (other attributes can be define and will ovrite the exciting values), Example:
````
project.set_artifact(key='training_model',artifact='<artifact object>',target_path='<target path>')#model artifacts
````

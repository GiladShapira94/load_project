# How To Load Project From Cluster A To Cluster B

#### prerequisite - 
* Git - 
  * You need to upload all the files that include in the project YAML on Cluster A to Git ot other remote storage (init_git=True,if True, will git init the context dir) 
  * Add to the function YAML attributes, with_repo = True - If the function is not a single file function, and it requires access to multiple files/libraries in the project.
* Remote Storage - 
  * All artifacts need to be upload to remote storage ot locally saved on Cluster B
* load project or create one, Example:
This important becuse you need to create an project object instance (MlrunProject object)
````
from os import path
import mlrun
import os

project_name_base = 'nyc-taxi-remote'

project = mlrun.get_or_create_project(name=project_name_base, user_project=True,init_git=True,context='./dev-project')
````
# Process on Cluster A (Source) - 
For this Example i allready run a project that called nyc-taxi and he had two functions (taxi: mlrun job, model-serving: mlrun serving) and one trained model.
## prerequisite - 
* Define a remote path for artifacts - 
  * You can change the path for each artifcats by change the artifact_path value in the log function, Examle:
  ````
  log_dataset(artifact_path='<Remote Path>','nyc-taxi-dataset-transformed', df=train_df,format='csv') 
  log_artifact(artifact_path='<Remote Path>','nyc-taxi-dataset-transformed' format='csv')    
  log_model('FareModel',body=dumps(model),artifact_path='<Remote Path>',model_file="FareModel.pkl")
  ````
  * You can change the project artifact path, this method will affect all fucntion (that mean that all artifacts will saved into this path), by using set_enviroment function, Example:
  ````
  artifact_path='s3://mlrun-v1-warroom/'
  mlrun.set_environment(artifact_path=artifact_path)
  ````
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
* This option allow to define which artifact object it related to.

### Set worfklow - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=set_workflow#mlrun.projects.MlrunProject.set_workflow)
Save workflow python file to Project YAML, Example:
* **Parameters  -**
  * name – name of the workflow
  * workflow_path – url/path for the workflow file
  * embed – add the workflow code into the project.yaml
  * engine – workflow processing engine (“kfp” or “local”)
  * args_schema – list of arg schema definitions (:py:class`~mlrun.model.EntrypointParam`)
  * handler – workflow function handler
  * args – argument values (key=value, ..) 

````
project.set_workflow('main', 'workflow.py', embed=True)
````
**Important -** After you stop edit the project YAML execute project.save() command to save to project YAML and export it.
````
project.save()
````

## Project YAML Example-
````
kind: project
metadata:
  name: nyc-taxi-gilad
  created: '2022-05-04T14:40:09.856000+00:00'
spec:
  functions:
  - url: taxi.yaml
    name: taxi
  - url: serving.py
    name: serving
    kind: nuclio
    image: mlrun/mlrun
  - url: trainer.py
    name: trainer
    image: mlrun/mlrun
    handler: train
  workflows:
  - name: main
    code: "from kfp import dsl\nfrom mlrun.platforms import auto_mount\nimport os\n\
      import sys\nimport mlrun\n\nfuncs = {}\n\n# init functions is used to configure\
      \ function resources and local settings\ndef init_functions(functions: dict,\
      \ project=None, secrets=None):\n    for f in functions.values():\n        f.apply(auto_mount())\n\
      \n@dsl.pipeline(\n    name=\"NYC Taxi Demo\",\n    description=\"Convert ML\
      \ script to MLRun\"\n)\n\ndef kfpipeline():\n\n    taxi_records_csv_path = mlrun.get_sample_path('data/Taxi/yellow_tripdata_2019-01_subset.csv')\n\
      \    zones_csv_path = mlrun.get_sample_path('data/Taxi/taxi_zones.csv')\n  \
      \  \n    # build our ingestion function (container image)\n    builder = funcs['taxi'].deploy_step(skip_deployed=True)\n\
      \    \n    # run the ingestion function with the new image and params\n    ingest\
      \ = funcs['taxi'].as_step(\n        name=\"fetch_data\",\n        handler='fetch_data',\n\
      \        image=builder.outputs['image'],\n        inputs={'taxi_records_csv_path':\
      \ taxi_records_csv_path,\n                'zones_csv_path': zones_csv_path},\n\
      \        outputs=['nyc-taxi-dataset', 'zones-dataset'])\n\n    # Join and transform\
      \ the data sets \n    transform = funcs[\"taxi\"].as_step(\n        name=\"\
      transform_dataset\",\n        handler='transform_dataset',\n        inputs={\"\
      taxi_records_csv_path\": ingest.outputs['nyc-taxi-dataset'],\n             \
      \   \"zones_csv_path\" : ingest.outputs['zones-dataset']},\n        outputs=['nyc-taxi-dataset-transformed'])\n\
      \n    # Train the model\n    train = funcs[\"taxi\"].as_step(\n        name=\"\
      train\",\n        handler=\"train_model\",\n        inputs={\"input_ds\" : transform.outputs['nyc-taxi-dataset-transformed']},\n\
      \        outputs=['FareModel'])\n    \n    # Deploy the model\n    deploy =\
      \ funcs[\"model-serving\"].deploy_step(models={\"taxi-serving_v1\": train.outputs['FareModel']},\
      \ tag='v2')\n"
    engine: null
  artifacts:
  - kind: model
    metadata:
      key: training_model
      project: nyc-taxi-gilad
      iter: 0
      tree: latest
      hash: 69f92d6005a0cdf9744ebf555e0b5f0ba0cfa5d4
    spec:
      target_path: v3io://webapi.default-tenant.app.cust-cs-3-2-3.iguazio-cd2.com/projects/fraud-demo-gilad/artifacts/model/3/model.pkl
      size: 27694
      db_key: training_model
      extra_data:
        probability calibration: v3io:///projects/fraud-demo-gilad/artifacts/model/plots/3/probability-calibration.html
        confusion matrix table.csv: v3io:///projects/fraud-demo-gilad/artifacts/model/3/confusion
          matrix table.csv
        confusion matrix: v3io:///projects/fraud-demo-gilad/artifacts/model/plots/3/confusion-matrix.html
        feature importances: v3io:///projects/fraud-demo-gilad/artifacts/model/plots/3/feature-importances.html
        feature importances table.csv: v3io:///projects/fraud-demo-gilad/artifacts/model/3/feature
          importances table.csv
        precision_recall_bin: v3io:///projects/fraud-demo-gilad/artifacts/model/plots/3/precision-recall-binary.html
        roc_bin: v3io:///projects/fraud-demo-gilad/artifacts/model/plots/3/roc-binary.html
      model_file: model.pkl
    status:
      state: created
  - kind: dataset
    metadata:
      project: nyc-taxi-gilad
      key: taxi-fetch_data_nyc-taxi-dataset
    spec:
      target_path: v3io://webapi.default-tenant.app.cust-cs-3-2-3.iguazio-cd2.com/projects/nyc-taxi-gilad/artifacts/data/nyc-taxi-dataset.csv
      format: ''
    status:
      state: created
  subpath: ''
  origin_url: ''
  load_source_on_run: true
  desired_state: online
  owner: Gilad
  disable_auto_mount: false
status:
  state: online
````
# Process on Cluster B (Target) -  
Now after you load the project YAML and upload to a remote storage all your relavent files you can load your project to the target cluster, in this example project files saved on GitHub excepte artifacts that saved on S3 Bucket. 


### Load Project YAML - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=load_project#mlrun.projects.load_project)
Load project YAML from GitHub, after excution the project will create with remote artifacts (not saved locally to V3IO) and not shown any functions later you will need to deploy or build each funciton.
* **Parameters  -**
  * context – project local directory path
  * url – name (in DB) or git or tar.gz or .zip sources archive path.
  * name – project name
  * secrets – key:secret dict or SecretsStore used to download sources
  * init_git – if True, will git init the context dir
  * subpath – project subpath (within the archive)
  * clone – if True, always clone (delete any existing content)
  * user_project – add the current user name to the project name (for db:// prefixes)
````
project = mlrun.load_project(context="./project",url="git://github.com/GiladShapira94/load_project.git",clone=True,init_git=True,name='load-project',user_project=True)
````
**Important -** if you change the project YAML for reload the project YAML you can use the reload function.
* **Parameters  -**
  * sync – set to True to load functions objects
  * context – context directory (where the yaml and code exist)
````
project.reload(sync=True)
````
#### Sync Project Functions - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=sync_functions#mlrun.projects.MlrunProject.sync_functions)
reload function objects from specs and files, after excution you will see the function on your project but it not deployed
````
project.sync_functions(save=True)
````
### Build fucntions - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=build_function#mlrun.projects.MlrunProject.build_function)
Deploying no remote function object such as MLRun jobs.
````
project.build_function('taxi')
````
### Deploy fucntions - [link to function documentation](https://docs.mlrun.org/en/latest/api/mlrun.projects.html?highlight=deploy_function#mlrun.projects.MlrunProject.deploy_function)
Deploying remote function object such as MLRun nuclio and serving.
**Important -** For getting function object, below you can see and example:.
````
project.get_function('taxi')
````

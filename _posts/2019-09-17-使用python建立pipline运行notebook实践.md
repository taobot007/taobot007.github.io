---
layout: post
title: 使用python建立pipline运行notebook实践
date: 2019-09-17
tag: 软件设计
---

## 1 启动
```python
# ignore warning
python -W ignore pipeline.py -n name1
```

## 2 pipline.py 接受输入参数，解读yaml file，并由papermill替换参数,保存到output path，并逐一启动notebook
```
import os
import glob
import time
import pprint
import argparse
import pickle
import numpy as np

from itertools import chain
from yaml import load, Loader, dump, Dumper
import papermill as pm
from hashids import Hashids

pp = pprint.PrettyPrinter(indent=2).pprint


def branch_rename_path(path, parameters):
    for brancher, branch in branching_rename.items():
        if brancher in path and branch in parameters:
            path = path.replace(brancher, parameters[branch])
    return path

def replace_base_path(params):
    for key, value in params.items():
        if type(value) is str and '+base_path+' in value:
            params[key] = value.replace('+base_path+', f"{params['base_path']}{params['runid']}/")
    return params                                    

def rename_parameter_paths(path_params, parameters):
    changed_paths_params = {}
    for key, value in path_params.items():
        if 'path' in key:
            changed_paths_params[key] = branch_rename_path(value, parameters)
        else:
            changed_paths_params[key] = value
    return changed_paths_params

def get_notebook_parameters(notebook_path):
    print(notebook_path)
    import json
    content = json.load(open(notebook_path, 'r'))
    parameter_cells = [i for i in content['cells'] if 'tags'in i['metadata'] and 'parameters' in i['metadata']['tags']]
    if len(parameter_cells) == 0:
        return []
    assert len(parameter_cells) == 1
    parameters = [line.split('=')[0].strip().split('#')[0] for line in parameter_cells[0]['source']]
    return [p for p in parameters if len(p) > 0]

def get_notebook_name(notebook_path):
    import json
    content = json.load(open(notebook_path, 'r'))
    return content['cells'][0]['source'][0]


def execute_plan(execution_plan, level=0):
    if type(execution_plan) == list:
        print("----------")
        for i, step in enumerate(execution_plan):
            print(f"Level {level}, Stage {i}")
            execute_plan(step, level+1)
        print("----------")
    else:
        (name, input_path, output_path, parameters) = execution_plan
        print(name)
        pm.execute_notebook(
            input_path=input_path,
            output_path=output_path,
            parameters=parameters
        )
        print(f"Finish {name}.")
        print()


def set_notebook_parameters(notebook_path, parameters_dict):
    wanted_parameters = get_notebook_parameters(notebook_path)
    return {p:parameters_dict[p] for p in wanted_parameters}


def simple_notebook_execution(runid, notebook, notebooks_output_folder, parameters):
    name = get_notebook_name(notebook)
    output_notebook = f"{notebooks_output_folder}{notebook.split('/')[-1]}"
    nb_parameters = set_notebook_parameters(notebook, parameters)
    return (name, notebook, branch_rename_path(output_notebook, parameters), nb_parameters)


def segmented_notebook_execution(runid, notebook, notebooks_output_folder, parameters):
    return [simple_notebook_execution(runid, notebook, notebooks_output_folder, parameters)]


def automated_execution_plan(runid, notebooks_input_folder, notebooks_output_folder, parameters):
    notebooks = sorted(glob.glob(f'{notebooks_input_folder}*'))

    execution_plan = []
    for notebook in notebooks:
        segmented_plan = segmented_notebook_execution(runid, notebook, notebooks_output_folder,
                                                          parameters)
        execution_plan = execution_plan + segmented_plan

    return execution_plan


def main(notebooks_input_path, notebooks_output_path, parameters_path,
         run_name=""):
    runid = Hashids().encode(int(time.time()))
    if run_name == "":
        notebooks_output_path = f"{notebooks_output_path}{runid}/"
    else:
        notebooks_output_path = f"{notebooks_output_path}{run_name}_{runid}/"

    os.makedirs(notebooks_output_path, exist_ok=True)
    print("Executing pipeline."
          f" Run ID: {runid}.")
    print(f"Output path: {notebooks_output_path}")

    print("Loading parameters...", end="")
    global_parameters = load(open(parameters_path), Loader=Loader)
    global_parameters['runid'] = runid
    global_parameters['run_name'] = run_name
    print("Done!")

    print("Loading execution plan...", end="")
    execution_plan =  automated_execution_plan(runid,
                                    notebooks_input_path,
                                    notebooks_output_path,
                                    global_parameters)

    pp(execution_plan)
    print("Done!")

    execution_plan_path = notebooks_output_path + "execution_plan.pickle"
    print(f"Saving execution plan on {execution_plan_path}.")
    pickle.dump(execute_plan, open(execution_plan_path, "wb"))
    print("Done!")

    parameters_output_path = notebooks_output_path + "parameters.yaml"
    print(f"Saving yaml of parameters on {parameters_output_path}.")
    dump(global_parameters, open(parameters_output_path, "w"), Dumper=Dumper)
    print("Done!")

    execute_plan(execution_plan)
    print(f"Run ID: {runid}.")
    print("Finished pipeline!")


def get_cmd_args():
    parser = argparse.ArgumentParser(description='Pipeline script')
    parser.add_argument("-i",
                        "--notebooks-input-folder",
                        help="The notebooks input folder.",
                        default="notebooks/main/",
                        type=str)
    parser.add_argument("-o",
                        "--notebooks-output-folder",
                        help="The notebooks output folder.",
                        default="notebooks/runs/",
                        type=str)
    parser.add_argument("-p",
                        "--parameters-path",
                        help="The parameters yaml path.",
                        default="parameters.yaml",
                        type=str)
    parser.add_argument("-n",
                        "--run-name",
                        help="If provided will replace the folder output folder name.",
                        default="",
                        type=str)
    args = parser.parse_args()
    return args


if __name__ == "__main__":
    args = get_cmd_args()
    res = main(args.input_notebooks_folder,
         args.output_notebooks_folder,
         args.parameters_path,
         args.run_name)
```

## 3 pickle the intermediate results

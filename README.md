# Electric Machine Optimization Tool (EMOT)
EMOT is a tool for optimal design of electric machines.

It consists of .aedt file  io for parsing model files from Ansys Maxwell and 
an optimization module from 
[Playtpus](https://github.com/Project-Platypus/Platypus)
in GitHub.


## .aedt file io

`AedtProject` can read and write .aedt file directly.

This project simply convert .aedt file to xml, load it using `xmltodict`.


## Installation
Install requirements first.

```shell
pip install -r requirements.txt
```


## Usage:

### Quick start

```python

from EMOT import AedtProject

model1 = AedtProject('aa.aedt', active_design='Maxwell2DDesign1')
model1.change_variables(
    design_name='Maxwell2DDesign1',
    var_name='x1',
    value='10mm'
)

# save to current file
model1.save()

# save file to another place or change name
model2 = model1.save_to(filename='a1.aedt')

# run simulation
model2.run_simulation(
    design_name='Maxwell2DDesign1',
    setup='Setup1',
    timeout_in_minutes=100
)
model2.export_csv()

```

### Generate models

```python

from EMOT import AedtProject
from EMOT.variables import StepReal

model1 = AedtProject('aa.aedt', active_design='Maxwell2DDesign1')
vars = {
    'to': StepReal(min_value=1, max_value=10, step=1, name='T0'),
    't1': StepReal(min_value=1, max_value=10, step=1, name='T1'),
}
combinations = model1.set_var_combination(vars)

# generate model
dataset_dir = model1.generate_models(
    path='./temp_models',
    var_combination=combinations
)

# collect data and form a dataset
dataset = model1.collect_data(dataset_dir)
```

### Tool for topology optimization

```python

from EMOT import AedtProject
import numpy as np
from TopologyModel import TopologyModel

model1 = AEDTProject('aa.aedt', active_design='Maxwell2DDesign1')

# initialize topology and generte dxf model
topology = TopologyModel('top1.npz')
dxf_model = topology.save_dxf('dxf_file.dxf')

# import dxf model to FEM model 
model1.import_dxf_and_subtract(dxf_model, symmetric=False)

# run simulation and export
model1.run_simulation(
    setup='Setup1',
    timeout_in_minutes=60
)

torque = model1.export_csv('torque', 'aa.csv')
field = model1.export_field('bxyz', 'field.fld')

# save field to npy
np.save('bxyz.npy', field)
```

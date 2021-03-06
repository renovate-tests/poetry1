# paranormal

A declarative, parameter-parsing library that provides multiple parsing interfaces (YAML, command line, and JSON) for loading parameters.

[![pypi version](https://img.shields.io/pypi/v/paranormal.svg)](https://pypi.org/project/paranormal/)

## Python Install:

Just install from PyPi using `pip install paranormal`.

## Running unit tests

Unit tests can be executed by running `pytest` from top folder of the repository.

## Using the Library

The following code samples show how this library is meant to be used.

### Subclass `Params`

```python
from paranormal.parameter_interface import *
from paranormal.params import *

# Note that only direct inheritance from Params is currently supported
class FrequencySweep(Params):
    """
    A frequency sweep measurement
    """
    freqs = SpanArangeParam(help='A list of frequencies to scan as [center, width, step]',
                            default=(1e9, 2e9, 0.1e9), unit='GHz')
    power = FloatParam(help='Power to transmit', default=-20, unit='dBm')
    pulse_samples = IntParam(help='Samples in the pulse', default=100)
    averages = IntParam(help='Number of sweeps to average over', default=10)
    is_test = BoolParam(help='Is this just a test', default=False)
    unseen_test = BoolParam(help='Hidden from the command line', default = False, hide=True)
    _unseen_test_2 = BoolParam(help='Hidden from the command line bc of the _', default=False)

```


### Reading From the Command Line

```python
parser = to_argparse(FrequencySweep)
# argparse will grab the command line arguments
# Ex command line: '--freqs 150 100 2 --power -40 --pulse_samples 200 --is_test'
args = parser.parse_args()
sweep_params = from_parsed_args(FrequencySweep, params_namespace=args)[0]

# even_simpler
sweep_params = create_parser_and_parse_args(FrequencySweep)
```

### Setting and getting parameters
```python
print(sweep_params.freqs)  # prints a numpy array of size (20,) array([0.0e+00, 1.0e+08, 2.0e+08, …
sweep_params.freqs = [5e9, 10e9, 2e9]
print(sweep_params.freqs)  # prints array([0.e+00, 2.e+09, 4.e+09, 6.e+09, 8.e+09])
print(sweep_params.is_test)  # prints True
sweep_params.freqs = [None, 10e9, 2e9]
print(sweep_params.freqs)  # prints [None, 10e9, 2e9]
```

### JSON and YAML serialization

```python
import json
import yaml  # pyyaml

d = to_json_serializable_dict(sweep_params)
s = json.dumps(d)
sweep_params = from_json_serializable_dict(json.loads(s))


to_yaml_file(sweep_params, 'test_params.yaml')
sweep_params = from_yaml_file('test_params.yaml')
```

### Nested Params
```python
class MultipleFreqSweeps(Params):
    sweep_1 = FrequencySweep(freqs=[0, 1e9, 0.1e9])  # overwrites default freqs value
    sweep_2 = FrequencySweep(freqs=[1e9, 2e9, 0.1e9])  # overwrites default freqs value
    
# Omit a nested param from the outer class:
class MultipleFreqSweepsHidden(Params):
    sweep_1 = FrequencySweep(freqs='__omit__')
    sweep_2 = FrequencySweep(power='__omit__')
        
# Customize prefixes used for command line parsing    
class MultipleFreqSweepsCustom(Params):
    sweep_1 = FrequencySweep()
    sweep_2 = FrequencySweep()
    __nested_prefixes__ = {'sweep_1': None, 'sweep_2': 'second_'}
    # Ex command line: '--freqs 100 120 2 --power -30 --second_power -40'
    
# Hide a nested param from the command line, but keep it in the class:
class MultipleFreqSweepsCustom(Params):
    sweep_1 = FrequencySweep()
    sweep_2 = FrequencySweep()
    __nested_prefixes__ = {'sweep_1': None, 'sweep_2': 'second_'}
    __params_to_hide__ = ['sweep_2_power']

```

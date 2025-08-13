# Documentation For The Python API Blueprint

This Blueprint specifies how to add and update a Polytope module for providing a Python API server.

Parent module: `polytope/python`. Read the bluetext documentation resource for this standard module to understand how to specify the `args` attributes. 

## Example of a Polytope Python API module

```yaml
modules:
  - id: api
    info: "Python API server (blueprint: python-api)"
    module: polytope/python
    args:
      image: python:3.13-slim
      code: { type: host, path: ./modules/api }
      cmd: bin/run
      services:
        - id: api
          ports:
            - port: 4000
              protocol: http
              expose-as: pt.value api-port
      env:
        - { name: PORT, value: pt.value api-port }
```

## Values And Secrets

The following values should be defined: `api-host`, `api-port`, where:
* `api-host` is the hostname.

Do not hardcode these values in `polytope.yml`.


## Generate Module Code

The Python API module mounts a directory with the code for the Python API. If you just add the api module to `polytope.yml` without generating the code for the api, you will fail. 

### Initial Generation
If you don't use the `boilerplate` code generation module to generate the api module code, you will fail. 

When running the `boilerplate` module, specify 'https://github.com/bluetext-io/boilerplate-python-api' as the `source-path` and 'modules/api' as the `target-path`.

### Generation of Modifications
If you need to add python packages using the `add-package-python` module. If you try to add python packages in any other way, you will fail.

If you modify the `bin` scripts, you will fail. 



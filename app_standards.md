### 1. Project structure ###
  * the directory normally called `app` or something similar should be named according to the project title, i.e. `records-acquisition-portal`
  * Scripts to run the server (i.e. manage.py and run.sh) should be in a scripts sub-directory
  * There should be argparse objects to all of the entry points
  *  init__.py should be named main.py (reserving init.py for things like logger config -- see below, and main.py for argument parsing and entry points.)
### 2. Dependency Management ###
  * we're currently using make files and conda to handle python dependencies in dev, and then makefiles + stdeb to build versioned debs
    * here's an example: https://github.com/drivergroup/pydriverutils/blob/master/Makefile
    * a more complex example: https://github.com/drivergroup/pipeline-bridge-service/blob/master/Makefile
    * another complex example: https://github.com/drivergroup/compbio-pipeline/blob/master/Makefile
    * the very near term plan is to move completely to using conda for python package dependencies, and . Here's a description in case your curious, https://conda.io/docs/building/build.html, but let's work on this together when you're in SF.
###3. Testing###
   * we're trying to move to solely using pytest
###4. Logging###
   `https://github.com/drivergroup/pydriverutils/blob/master/pydriverutils/driver_logging.py` -- has logging functions. You can import the relevant methods with (for example):
```python
from pydriverutils.logging import configure_root_logger_from_args, build_log_parser
```
The pattern is:
```python
import argparse
parser = argparse.ArgumentParser(parents=[build_log_parser()])
```
.... add other arguments ...
```python
args=parser.parse_args()
configure_root_logger_from_args(args)
```
By default, this configures the root logger info and above to stderr. You can pass additional arguments if you want to further customize the parsing. Furthermore, you can pass additional arguments on the command line (e.g. --debug) to change the logging verbosity, and a file output at a different verbosity, etc.
Then, in the base init function, write:
```python
import logging
from .driver_logging import Logger
logging.setLoggerClass(Logger)
```
this sets the standard logging class to use our extended logger (which has functionality for allowing popen calls to write to a logger)
Finally, in each script, you can run:
```python
logger = logging.getLogger(name)
```
and that should do the right thing (just like you're already doing)
### 5. DRIVER API ### 
There's a very young python interface to the DRIVER API in pydriverutils -- we should move all of the code from driver_api.py into there to facilitiate re-use. We also have a MockDriverAPI object in pydriverutils which has been very useful for testing the interfaces. We should do the same for the new methods that you add.

### 6. Design elements & CSS ###
We've been using flask_admin for the navigation. We should either convert RAP to use the same, or convert CAT and the IHC portal to use Blueprint.
EDIT: after sleeping on it, I like this templating structure better than flask_admin. I'll convert ihc_portal and add a JIRA ticket for CAT.
Also, we need to consolidate the CSS and document and choose a consistent naming scheme. Again, I'm not married to CAT's CSS but I do think that we should be consistent across projects.
### 7. Configuration and Environment ###
Since we're using environment variables in production, we've standardized on using .env files for configuration management. Here's an example: https://github.com/drivergroup/sample-accessioning-bridge/blob/master/sample_accessioning_bridge/main.py
#### In short: ####
1. add a --env option to the argument parser
2. use pydriverutils environment module to load the .env file, taking environment variables from the environment by default
keep a dev .env file in an environments directory in the project base, which should allow the app to be tested locally

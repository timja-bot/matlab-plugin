# Plugin Configuration Guide

When you define an automated pipeline of tasks in Jenkins&trade;, whether in the web UI or with a [`Jenkinsfile`](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/), you can use the plugin to run your MATLAB&reg; code or Simulink&reg; models. This guide demonstrates how to configure the plugin and use it in freestyle, multi-configuration, and pipeline projects.

> :information_source: **Note:** To run MATLAB code or Simulink models, Jenkins requires a valid MATLAB or Simulink license. If you have installed Jenkins as a Windows&reg; service application, you need to provide a MathWorks&reg; username and password in the **Jenkins Properties** window (accessible from within the Windows Task Manager).

-  [Configure Plugin in Web UI](#configure-plugin-in-web-ui)
      -  [Use MATLAB in Build](#use-matlab-in-build)
      -  [Specify Build Steps](#specify-build-steps)
         - [Run MATLAB Build](#run-matlab-build)
         - [Run MATLAB Tests](#run-matlab-tests)
         - [Run MATLAB Command](#run-matlab-command)
-  [Set Up Freestyle Project](#set-up-freestyle-project)
-  [Set Up Multi-Configuration Project](#set-up-multi-configuration-project)
      -  [Add MATLAB Axis](#add-matlab-axis)
      -  [Add User-Defined Axis](#add-user-defined-axis)
-  [Set Up Pipeline Project](#set-up-pipeline-project)
   -  [Add MATLAB to System Path](#add-matlab-to-system-path)
   -  [Use the runMATLABBuild Step](#use-the-runmatlabbuild-step)
   -  [Use the runMATLABTests Step](#use-the-runmatlabtests-step)
   -  [Use the runMATLABCommand Step](#use-the-runmatlabcommand-step) 
   -  [Use MATLAB in Matrix Build](#use-matlab-in-matrix-build)
-  [Register MATLAB as Jenkins Tool](#register-matlab-as-jenkins-tool)
   -  [Use MATLAB as a Tool in Pipeline Project](#use-matlab-as-a-tool-in-pipeline-project)

## Configure Plugin in Web UI
You can use the web UI provided by Jenkins to configure the plugin in freestyle and multi-configuration projects. To run MATLAB or Simulink in a pipeline project, see [Set Up Pipeline Project](#set-up-pipeline-project).

### Use MATLAB in Build
Once you install the plugin, **Use MATLAB version** appears in the **Build Environment** section of the project configuration window. Select **Use MATLAB version** to specify the MATLAB version you want to use in the build. You can select one of the MATLAB versions that have been registered as Jenkins tools, or you can select **Custom** if you want to specify a different version. For more information about registering a MATLAB version as a tool, see [Register MATLAB as Jenkins Tool](#register-matlab-as-jenkins-tool).

In this example, the list includes two registered tools as well as the option for specifying a custom installation. If you select **Custom**, a **MATLAB root** box appears in the UI. You must enter the full path to your preferred MATLAB root folder in this box.

![use_matlab_version_tool](https://user-images.githubusercontent.com/48831250/217575011-ab82e8bf-c673-4364-946a-cdd54eb75c39.png)

When you specify a MATLAB version in the **Build Environment** section, the plugin prepends its `bin` folder to the system PATH environment variable of the build agent and invokes it to perform the build. If the build agent already has your preferred MATLAB version on the path, then you are not required to select **Use MATLAB version**. In this case, the plugin uses the topmost MATLAB version on the system path. The build fails if the operating system cannot find MATLAB on the path.

You can use the [`matlabroot`](https://www.mathworks.com/help/matlab/ref/matlabroot.html) function to return the full path to your preferred MATLAB root folder. The path depends on the platform, MATLAB version, and installation location. This table shows examples of the root folder path on different platforms. 

| Platform     | Path to MATLAB Root Folder      |
|--------------|---------------------------------|
| Windows      | C:\Program Files\MATLAB\R2022b  |
| Linux&reg;   | /usr/local/MATLAB/R2022b        |
| macOS        | /Applications/MATLAB_R2022b.app |

### Specify Build Steps
When you set up the **Build Steps** section of the project configuration window, the plugin provides you with three build steps:

* To run a MATLAB build, use the [Run MATLAB Build](#run-matlab-build) step.
* To run MATLAB and Simulink tests and generate artifacts, use the [Run MATLAB Tests](#run-matlab-tests) step.
* To run a MATLAB script, function, or statement, use the [Run MATLAB Command](#run-matlab-command) step.

If you use a source code management (SCM) system such as Git&trade;, then your project should include the appropriate SCM configuration to check out the code before it can invoke the plugin. If you do not use any SCM systems to manage your code, then an additional build step might be required to ensure that the code is available in the project workspace before the build starts.

#### Run MATLAB Build
The **Run MATLAB Build** step lets you run a build using the [MATLAB build tool](https://www.mathworks.com/help/matlab/matlab_prog/overview-of-matlab-build-tool.html). You can use this step to run the tasks specified in a file named `buildfile.m` in the root of your repository. To use the **Run MATLAB Build** step, you need MATLAB R2022b or a later release.

Specify the tasks you want to execute in the **Tasks** box. If you specify more than one task, use a space to separate them. If you do not specify any tasks, the plugin runs the default tasks in `buildfile.m` as well as all the tasks on which they depend. For example, enter `mytask` in the **Tasks** box to run a task named `mytask` as well as all the tasks on which it depends.

![run_matlab_build](https://user-images.githubusercontent.com/48831250/217647265-38b0530f-f9f4-43a7-91ef-719e8d0a545e.png)

MATLAB exits with exit code 0 if the specified tasks run successfully. Otherwise, MATLAB terminates with a nonzero exit code, which causes the Jenkins build to fail.

When you use this step, a file named `buildfile.m` must be in the root of your repository. For more information about the build tool, see [Create and Run Tasks Using Build Tool](https://www.mathworks.com/help/matlab/matlab_prog/create-and-run-tasks-using-build-tool.html).

#### Run MATLAB Tests
The **Run MATLAB Tests** build step lets you run MATLAB and Simulink tests and generate artifacts such as test results in JUnit XML format and code coverage results in Cobertura XML format. By default, the plugin includes any test files in your [MATLAB project](https://www.mathworks.com/help/matlab/projects.html) that have a `Test` label. If your build does not use a MATLAB project, or if it uses a MATLAB release before R2019a, then the plugin includes all tests in the root of your repository and in any of its subfolders.
 
You can customize the **Run MATLAB Tests** build step in the step configuration interface. For example, you can add source folders to the MATLAB search path, control which tests to run, and generate various test and coverage artifacts. If you do not select any of the existing options, all the tests in your project run, and any test failure causes the build to fail.
 
Select **Source folder** if you want to specify the location of a folder containing source code, relative to the project root folder. The plugin adds the specified folder and its subfolders to the top of the MATLAB search path. If you specify a source folder and then generate coverage results, the plugin uses only the source code in the specified folder and its subfolders to generate the results. You can specify more than one folder by clicking **Add folder**.

![run_matlab_tests_source](https://user-images.githubusercontent.com/48831250/217649842-2791c7e4-2fb9-4031-b4ef-1a3c16f082e0.png)

By default, the **Run MATLAB Tests** step creates a test suite from all the tests in your project. To create a filtered test suite, select **By folder name**, **By tag**, or both:

* Select **By folder name** if you want to specify the location of a folder containing test files, relative to the project root folder. The plugin creates a test suite using only the tests in the specified folder and its subfolders. You can specify more than one folder by clicking **Add folder**.
* Select **By tag** if you want to select test suite elements using a specified test tag.

![run_matlab_tests_filter](https://user-images.githubusercontent.com/48831250/217650500-bebac243-ec5e-4b21-a4c3-340961005780.png)

To customize your test run, select options in the **Customize Test Run** section:

* To apply strict checks when running the tests, select **Strict**. If you select this option, the plugin generates a qualification failure whenever a test issues a warning. Selecting **Strict** is the same as specifying the `Strict` name-value argument of the [`runtests`](https://www.mathworks.com/help/matlab/ref/runtests.html) function as `true`.
* To run tests in parallel, select **Use parallel**. Selecting **Use parallel** is the same as specifying the `UseParallel` name-value argument of `runtests` as `true`. You must have Parallel Computing Toolbox&trade; installed to use this option. If other selected options are not compatible with running tests in parallel, the plugin runs the tests in serial regardless of your selection.
* To control the amount of output detail displayed for your test run, select a value from the **Output detail** list. Selecting a value for this option is the same as specifying the `OutputDetail` name-value argument of `runtests` as that value. By default, the plugin displays failing and logged events at the `Detailed` level and test run progress at the `Concise` level.
* To include diagnostics logged by the [`log (TestCase)`](https://www.mathworks.com/help/matlab/ref/matlab.unittest.testcase.log.html) and [`log (Fixture)`](https://www.mathworks.com/help/matlab/ref/matlab.unittest.fixtures.fixture.log.html) methods at a specified verbosity level, select a value from the **Logging level** list. Selecting a value for this option is the same as specifying the `LoggingLevel` name-value argument of `runtests` as that value. By default, the plugin includes diagnostics logged at the `Terse` level. 

![run_matlab_tests_customization](https://user-images.githubusercontent.com/48831250/217650823-3493b335-ef28-4b26-8516-0334788956ef.png)

To generate test and coverage artifacts, select options in the **Generate Test Artifacts** and **Generate Coverage Artifacts** sections. To publish the test results, you can use these artifacts with other Jenkins plugins. By default, the plugin assigns a name to each selected artifact and stores it in the `matlabTestArtifacts` folder of the project workspace. You can override the default artifact name and location by specifying a path relative to the project root folder in the **File path** box. The plugin does not create the `matlabTestArtifacts` folder if the name of the folder does not appear in any of the displayed **File path** boxes.

![run_matlab_tests_artifacts](https://user-images.githubusercontent.com/48831250/217651806-1c8a6e9a-13a8-4d05-819b-e97533cc7bae.png)

The **Run MATLAB Tests** build step produces a MATLAB script file and uses it to run the tests and generate the artifacts. The plugin writes the contents of this file to the build log. You can review the build log in **Console Output** to understand the testing workflow.
 
Artifacts to generate with the plugin are subject to these restrictions: 
* Producing a PDF test report on macOS platforms is supported in MATLAB R2020b and later.
* Exporting Simulink Test&trade; Manager results requires a Simulink Test license and is supported in MATLAB R2019a and later.
* Collecting model coverage results requires a Simulink Coverage&trade; license and is supported in MATLAB R2018b and later.

#### Run MATLAB Command
The **Run MATLAB Command** build step lets you run MATLAB scripts, functions, and statements. You can use this build step to customize your test run or execute any MATLAB commands.

Specify the MATLAB script, function, or statement you want to execute in the **Command** box. If you specify more than one script, function, or statement, use a comma or semicolon to separate them. If you want to run a script or function, do not specify the file extension.

For example, enter `myscript` in the **Command** box to run a script named `myscript.m` in the root of your repository.

![run_matlab_command](https://user-images.githubusercontent.com/48831250/217652304-8f7b351a-c52f-4f2b-a911-f2bc4c94dec0.png)

MATLAB exits with exit code 0 if the specified script, function, or statement executes successfully without error. Otherwise, MATLAB terminates with a nonzero exit code, which causes the build to fail. To fail the build in certain conditions, use the [`assert`](https://www.mathworks.com/help/matlab/ref/assert.html) or [`error`](https://www.mathworks.com/help/matlab/ref/error.html) functions.

When you use this step, all the required files must be on the MATLAB search path. If your script or function is not in the root of your repository, you can use the [`addpath`](https://www.mathworks.com/help/matlab/ref/addpath.html), [`cd`](https://www.mathworks.com/help/matlab/ref/cd.html), or [`run`](https://www.mathworks.com/help/matlab/ref/run.html) functions to ensure that it is on the path when invoked. For example, to run `myscript.m` in a folder named `myfolder` located in the root of the repository, you can specify the contents of the **Command** box like this:

`addpath("myfolder"), myscript`

## Set Up Freestyle Project
To configure the plugin for a freestyle project, specify the MATLAB version to use as well as the required build steps.

To specify the MATLAB version, select **Use MATLAB version** in the **Build Environment** section of the project configuration window. Then, specify the MATLAB version that Jenkins should use in the build. You can skip this step if MATLAB has already been added to the path on the build agent.

![build_environment](https://user-images.githubusercontent.com/48831250/217673448-961f14de-6985-453e-944f-96b9ffe29b99.png)

To run MATLAB code and Simulink models, specify the appropriate build steps in the **Build Steps** section:
* If you add the [**Run MATLAB Build**](#run-matlab-build) step, specify your MATLAB build tasks in the **Tasks** box. 
* If you add the [**Run MATLAB Tests**](#run-matlab-tests) step, specify your source code, test suite filters, run customization options, and test and coverage artifacts to be generated in the project workspace.
* If you add the [**Run MATLAB Command**](#run-matlab-command) step, specify your MATLAB script, function, or statement in the **Command** box. 

  
## Set Up Multi-Configuration Project
The plugin supports [multi-configuration (matrix) projects](https://plugins.jenkins.io/matrix-project/). Multi-configuration projects are useful when builds include similar steps. For example:
* The same test suite runs on different platforms (such as Windows, Linux, and macOS).
* The same test suite runs against different versions of MATLAB.

To configure the plugin for a multi-configuration project, specify the MATLAB versions to use as well as the required build steps. You can add axes in the **Configuration Matrix** section to specify the duplicating build steps. 

There are two ways to specify multiple MATLAB versions in a multi-configuration project: using the **MATLAB** axis or using a user-defined axis.

![add_axis](https://user-images.githubusercontent.com/48831250/217654844-0b35bf40-063d-4f1e-a45a-d2fd23f19c07.png)

### Add MATLAB Axis
If your Jenkins instance includes MATLAB versions registered as tools, then **MATLAB** appears as an option when you click **Add axis** in the **Configuration Matrix** section. By adding the **MATLAB** axis, you can select MATLAB versions and add them as axis values to your matrix configuration. The list includes all MATLAB versions that have been registered as Jenkins tools. In this example, there are two MATLAB versions registered as tools. In each build iteration, the plugin prepends one of the selected versions to the PATH environment variable and invokes it to run the build. 

![matlab_axis](https://user-images.githubusercontent.com/48831250/217655206-0fdade64-17f5-4604-a730-50b90871e939.png)

For more information about registering a MATLAB version as a tool, see [Register MATLAB as Jenkins Tool](#register-matlab-as-jenkins-tool).

> :information_source: **Note:** When you add the **MATLAB** axis, do not select **Use MATLAB version**. Any values you specify by **Use MATLAB version** take precedence over the values specified by the **MATLAB** axis.

### Add User-Defined Axis
If you do not specify the **MATLAB** axis, add a user-defined axis in the **Configuration Matrix** section to specify the MATLAB versions in the build. Enter the name of the axis in the **Name** box and its values in the **Values** box. Separate the values with a space. For instance, specify two MATLAB versions to run the same set of tests.

![user_defined_axis](https://user-images.githubusercontent.com/48831250/217655769-5e1c19be-372f-460c-9c11-456dc749e142.png)

When you add a user-defined axis to specify MATLAB versions, you must also specify where they are installed. To do this, select **Use MATLAB version** in the **Build Environment** section and then construct a root folder path using the axis name. In this example, `$VERSION` in the **MATLAB root** box is replaced by one axis value per build iteration. 

![build_environment_matrix](https://user-images.githubusercontent.com/48831250/217656233-4b48181f-4236-4bb4-9a28-20cc119fb859.png)

A multi-configuration project creates a separate workspace for each user-defined axis value. If you specify the full paths to where different MATLAB versions are installed as axis values, the plugin fails to create separate workspaces and fails the build.

> :information_source: **Note:** Both `$VAR` and `${VAR}` are valid formats for accessing the values of the axis `VAR`. On macOS platforms, the `${VAR}` format is recommended.

You can add several axes in the **Configuration Matrix** section. For example, add the **MATLAB** axis to specify MATLAB versions and the user-defined `TEST_TAG` axis to specify the test tags for a group of tests.

![axis_matlab_testtag](https://user-images.githubusercontent.com/48831250/217656693-e01f8c6c-dd10-481e-9d1d-f32c48ac9365.png)

Once you have specified the axes, add the required build steps in the **Build Steps** section:

* If you add the [**Run MATLAB Build**](#run-matlab-build) step, specify your MATLAB build tasks in the **Tasks** box.

* If you add the [**Run MATLAB Tests**](#run-matlab-tests) step, specify your source code, test suite filters, run customization options, and test and coverage artifacts to be generated in the project workspace.

* If you add the [**Run MATLAB Command**](#run-matlab-command) step, specify your MATLAB script, function, or statement in the **Command** box. You can use the user-defined axes to specify the contents of the **Command** box. For example:
   
  ```
  results = runtests(pwd,"Tag","$TEST_TAG"); assertSuccess(results);
  ```

## Set Up Pipeline Project
When you define your pipeline with a `Jenkinsfile`, the plugin provides you with three build steps:

* To run a MATLAB build, use the [`runMATLABBuild`](#use-the-runmatlabbuild-step) step.
* To run MATLAB and Simulink tests and generate artifacts, use the [`runMATLABTests`](#use-the-runmatlabtests-step) step.
* To run a MATLAB script, function, or statement, use the [`runMATLABCommand`](#use-the-runmatlabcommand-step) step.

To configure the plugin for a pipeline project:
1) Define your pipeline in a `Jenkinsfile` in the root of your repository.
2) In the **Pipeline** section of the project configuration window, select **Pipeline script from SCM** from the **Definition** list. 
3) Select your source control system from the **SCM** list.
4) Paste your repository URL into the **Repository URL** box.

You also can define your pipeline directly in the project configuration window. If you select **Pipeline script** from the **Definition** list, you can author your pipeline code in the **Script** box. When you define your pipeline this way, it must include an additional stage to check out your code from source control.

### Add MATLAB to System Path
When the plugin executes MATLAB related steps in your pipeline, it uses the topmost MATLAB version on the system path. If the PATH environment variable of the build agent does not include any MATLAB versions, you must update the variable with the MATLAB root folder that should be used for the build.

To update the system PATH environment variable using declarative pipeline syntax, use an `environment` block in your `Jenkinsfile`. For example, prepend MATLAB R2022b to the system PATH environment variable and use it to run your command.

```groovy
// Declarative Pipeline
pipeline {
   agent any
   environment {
       PATH = "C:\\Program Files\\MATLAB\\R2022b\\bin;${PATH}"   // Windows agent
    // PATH = "/usr/local/MATLAB/R2022b/bin:${PATH}"   // Linux agent
    // PATH = "/Applications/MATLAB_R2022b.app/bin:${PATH}"   // macOS agent    
   }
    stages {
        stage('Run MATLAB Command') {
            steps {
               runMATLABCommand "disp('Hello World!')"
            }       
        }                
    } 
}
``` 

If you define your pipeline using scripted pipeline syntax, set the PATH environment variable in the `node` block. For example: 

```groovy
// Scripted Pipeline
node {
    env.PATH = "C:\\Program Files\\MATLAB\\R2022b\\bin;${env.PATH}"   //Windows agent
    // env.PATH = "/usr/local/MATLAB/R2022b/bin:${env.PATH}"   //Linux agent
    // env.PATH = "/Applications/MATLAB_R2022b.app/bin:${env.PATH}"   //macOS agent
    runMATLABCommand "disp('Hello World!')"
}
``` 

### Use the `runMATLABBuild` Step
Use the `runMATLABBuild` step in your pipeline to run a build using the [MATLAB build tool](https://www.mathworks.com/help/matlab/matlab_prog/overview-of-matlab-build-tool.html). You can use this step to run the tasks specified in a file named `buildfile.m` in the root of your repository. To use the `runMATLABBuild` step, you need MATLAB R2022b or a later release.

The `runMATLABBuild` step accepts an optional input. Use this input to specify the tasks to run. If you specify the step in your `Jenkinsfile` without an input (`runMATLABBuild()`), the plugin runs the default tasks in `buildfile.m` as well as all the tasks on which they depend.

Input                     | Description    
------------------------- | ---------------
`tasks`                   | (Optional) Tasks to execute. If you specify more than one task, use a space to separate them.<br/>**Example:** `'mytask'`<br/>**Example:** `'compile test'`

For example, in your `Jenkinsfile`, define a declarative pipeline to run a task named `mytask` as well as all the tasks on which it depends.

```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('Run MATLAB Build') {
            steps {
                runMATLABBuild(tasks: 'mytask')
            }       
        }                
    } 
}
``` 

You also can use `runMATLABBuild` in a scripted pipeline.

```groovy
// Scripted Pipeline
node {
    runMATLABBuild(tasks: 'mytask')
}
``` 

MATLAB exits with exit code 0 if the specified tasks run successfully. Otherwise, MATLAB terminates with a nonzero exit code, which causes the stage to fail.

When you use this step, a file named `buildfile.m` must be in the root of your repository. For more information about the build tool, see [Create and Run Tasks Using Build Tool](https://www.mathworks.com/help/matlab/matlab_prog/create-and-run-tasks-using-build-tool.html).


### Use the `runMATLABTests` Step
Use the `runMATLABTests` step in your pipeline to run MATLAB and Simulink tests and generate various test and coverage artifacts. By default, the plugin includes any test files in your [MATLAB project](https://www.mathworks.com/help/matlab/projects.html) that have a `Test` label. If your pipeline does not use a MATLAB project, or if it uses a MATLAB release before R2019a, then the plugin includes all tests in the root of your repository and in any of its subfolders.

For example, in your `Jenkinsfile`, define a declarative pipeline to run the tests in your project.


```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('Run MATLAB Tests') {
            steps {
                runMATLABTests()
            }       
        }                
    } 
}
``` 

Use the `runMATLABTests` step in a scripted pipeline to run the tests in your project.

```groovy
// Scripted Pipeline
node {
    runMATLABTests()  
}
``` 

MATLAB exits with exit code 0 if the test suite runs successfully without any test failures. Otherwise, MATLAB terminates with a nonzero exit code, which causes the stage to fail.

You can customize the `runMATLABTests` step using optional inputs. For example, you can add source folders to the MATLAB search path, control which tests to run, and generate various artifacts.

Input                     | Description    
------------------------- | ---------------
`sourceFolder`            | (Optional) Location of the folder containing source code, relative to the project root folder. The specified folder and its subfolders are added to the top of the MATLAB search path. If you specify `sourceFolder` and then generate coverage results, the plugin uses only the source code in the specified folder and its subfolders to generate the results. You can specify multiple folders using a comma-separated list.<br/>**Example:** `['source']`<br/>**Example:** `['source/folderA', 'source/folderB']`
`selectByFolder`          | (Optional) Location of the folder used to select test suite elements, relative to the project root folder. To create a test suite, the plugin uses only the tests in the specified folder and its subfolders. You can specify multiple folders using a comma-separated list.<br/>**Example:** `['test']`<br/>**Example:** `['test/folderA', 'test/folderB']`
`selectByTag`             | (Optional) Test tag used to select test suite elements. To create a test suite, the plugin uses only the test elements with the specified tag.<br/>**Example:** `'FeatureA'`
`strict`                  | (Optional) Whether to apply strict checks when running tests, specified as `false` or `true`. By default, the value is `false`. If you specify a value of `true`, the plugin generates a qualification failure whenever a test issues a warning.
`useParallel`             | (Optional) Whether to run tests in parallel, specified as `false` or `true`. By default, the value is `false` and tests run in serial. If the test runner configuration is suited for parallelization, you can specify a value of `true` to run tests in parallel. This input requires a Parallel Computing Toolbox license.
`outputDetail`            | (Optional) Amount of output detail displayed for the test run, specified as `'None'`, `'Terse'`, `'Concise'`, `'Detailed'`, or `'Verbose'`. By default, the plugin displays failing and logged events at the `Detailed` level and test run progress at the `Concise` level.
`loggingLevel`            | (Optional) Maximum verbosity level for logged diagnostics included for the test run, specified as `'None'`, `'Terse'`, `'Concise'`, `'Detailed'`, or `'Verbose'`. By default, the plugin includes diagnostics logged at the `Terse` level. 
`testResultsPDF`          | (Optional) Path to write the test results in PDF format. On macOS platforms, this input is supported in MATLAB R2020b and later.<br/>**Example:** `'test-results/results.pdf'`      
`testResultsTAP`          | (Optional) Path to write the test results in TAP format.<br/>**Example:** `'test-results/results.tap'`
`testResultsJUnit`        | (Optional) Path to write the test results in JUnit XML format.<br/>**Example:** `'test-results/results.xml'`
`testResultsSimulinkTest` | (Optional) Path to export Simulink Test Manager results in MLDATX format. This input requires a Simulink Test license and is supported in MATLAB R2019a and later.<br/>**Example:** `'test-results/results.mldatx'`
`codeCoverageCobertura`   | (Optional) Path to write the code coverage results in Cobertura XML format.<br/>**Example:** `'code-coverage/coverage.xml'`
`modelCoverageCobertura`  | (Optional) Path to write the model coverage results in Cobertura XML format. This input requires a Simulink Coverage license and is supported in MATLAB R2018b and later.<br/>**Example:** `'model-coverage/coverage.xml'`

For instance, define a declarative pipeline to run the tests in your MATLAB project, and then generate test results in JUnit XML format and code coverage results in Cobertura XML format at specified locations on the build agent. Generate the coverage results for only the code in the `source` folder in the root of your repository. 


```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('Run MATLAB Tests') {
            steps {
                runMATLABTests(testResultsJUnit: 'test-results/results.xml',
                               codeCoverageCobertura: 'code-coverage/coverage.xml',
                               sourceFolder: ['source'])
            }       
        }                
    } 
}
``` 

Define a scripted pipeline to run your tests and generate artifacts.

```groovy
// Scripted Pipeline
node {
    runMATLABTests(testResultsJUnit: 'test-results/results.xml',
                   codeCoverageCobertura: 'code-coverage/coverage.xml',
                   sourceFolder: ['source']) 
}
``` 

### Use the `runMATLABCommand` Step
Use the `runMATLABCommand` step in your pipeline to run MATLAB scripts, functions, and statements. You can use this step to customize your test run or execute any MATLAB commands.

You must provide `runMATLABCommand` with a string that specifies the script, function, or statement you want to execute. If you specify more than one script, function, or statement, use a comma or semicolon to separate them. If you want to run a script or function, do not specify the file extension.

**Example:** `runMATLABCommand 'myscript'`<br/>
**Example:** `runMATLABCommand 'results = runtests, assertSuccess(results);'` 

For example, in your `Jenkinsfile`, define a declarative pipeline to run a script named `myscript.m`.

```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('Run MATLAB Command') {
            steps {
                runMATLABCommand 'myscript'
            }       
        }                
    } 
}
``` 

You also can use `runMATLABCommand` in a scripted pipeline.

```groovy
// Scripted Pipeline
node {
    runMATLABCommand 'myscript'  
}
``` 

MATLAB exits with exit code 0 if the specified script, function, or statement executes successfully without error. Otherwise, MATLAB terminates with a nonzero exit code, which causes the stage to fail. If you properly react to the resulting MATLAB execution exception, the remaining stages of your pipeline can still run, and your build can succeed. Otherwise, Jenkins terminates the build in the current stage and marks it as a failure.

When you use the `runMATLABCommand` step, all the required files must be on the MATLAB search path. If your script or function is not in the root of your repository, you can use the [`addpath`](https://www.mathworks.com/help/matlab/ref/addpath.html), [`cd`](https://www.mathworks.com/help/matlab/ref/cd.html), or [`run`](https://www.mathworks.com/help/matlab/ref/run.html) functions to ensure that it is on the path when invoked. For example, to run `myscript.m` in a folder named `myfolder` located in the root of the repository, you can specify the `runMATLABCommand` step like this: 

`runMATLABCommand 'addpath("myfolder"), myscript'` 

## Use MATLAB in Matrix Build
Like multi-configuration projects, you can use MATLAB as part of a [matrix](https://www.jenkins.io/doc/book/pipeline/syntax/#declarative-matrix) build in pipeline projects. For example, you can define a pipeline to run your test suite on different platforms or against different versions of MATLAB.

This example defines a declarative pipeline to run your MATLAB code and generate artifacts using MATLAB R2021b, R2022a, and R2022b. The pipeline has a `matrix` block to define the possible name-value combinations that should run in parallel. 

```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('BuildAndTest') {
            matrix {
                agent any
                environment {
                    PATH = "C:\\Program Files\\MATLAB\\${MATLAB_VERSION}\\bin;${PATH}"   // Windows agent
                }
                axes {
                    axis {
                        name 'MATLAB_VERSION'
                        values 'R2021b', 'R2022a', 'R2022b'
                    }
                }
                stages {
                    stage('Run MATLAB commands') {
                        steps {
                            runMATLABCommand 'ver, pwd'
                        }
                    }
                    stage('Run MATLAB tests') {
                        steps {
                            runMATLABTests(testResultsJUnit: 'test-results/results.xml',
                                           codeCoverageCobertura: 'code-coverage/coverage.xml')
                        }  
                    }
                }
            } 
        }
    }
}
``` 

## Register MATLAB as Jenkins Tool
When you run MATLAB code and Simulink models as part of your automated pipeline of tasks, Jenkins invokes MATLAB as an external program. When you configure your project, you can explicitly specify the MATLAB version that Jenkins should invoke by providing the path to the preferred MATLAB root folder. For example, you can use an `environment` block in your `Jenkinsfile` to specify a MATLAB root folder for your pipeline project.

Instead of specifying the path to the MATLAB root folder on a per-project basis, you can register a MATLAB version as a Jenkins tool, which makes it available to any project you configure in Jenkins. Once you have registered a MATLAB version as a tool, you no longer need to specify its root folder path within a project. Jenkins only needs the tool name to access the MATLAB version.

To register a MATLAB version as a Jenkins tool:

1) In your Jenkins interface, select **Manage Jenkins > Global Tool Configuration**. The **Global Tool Configuration** page opens where you can register different tools with Jenkins.
2) In the **MATLAB** section of the **Global Tool Configuration** page, click **Add MATLAB**. The section expands and lets you assign a name to your preferred MATLAB version and specify its installation location.
3) Specify the name you want to assign to the MATLAB version in the **Name** box, and enter the full path to its root folder in the **MATLAB root** box. To register the MATLAB version as a tool, do not select **Install automatically**. 
4) To confirm your choices, click **Save** at the bottom of the page.

For example, register MATLAB R2022b as a Jenkins tool on your Windows local agent.

![matlab_tool](https://user-images.githubusercontent.com/48831250/217572380-21013b39-f85b-4709-a39d-3f862bb9d1eb.png)

If your Jenkins instance includes remote agents, you can register MATLAB as a tool on the remote agents using the tool name that you have specified on the local agent. For example, if you have registered MATLAB R2022b as a tool on your local agent, you can register the same MATLAB version installed on a remote agent as a tool on that agent. To register a MATLAB version as a Jenkins tool on a remote agent: 

1) Navigate to the **Node Properties** interface of the agent. You can access this interface by selecting **Manage Jenkins > Manage Nodes and Clouds**, following the link corresponding to the agent, and then selecting **Configure** on the left.
2) Select **Tool Locations**. Then, select the tool name from the **Name** list. The list contains the names assigned to the registered MATLAB versions on the local agent.  
3) In the **Home** box, enter the full path to the MATLAB root folder on the remote agent.
4) Click **Save** to confirm your choices.

### Use MATLAB as a Tool in Pipeline Project
To invoke MATLAB as a Jenkins tool using declarative pipeline syntax, use a `tools` block in your `Jenkinsfile`. To specify the tool in the block, use the `matlab` keyword followed by the name assigned to the tool on the **Global Tool Configuration** page. For example, run `myscript.m` using the MATLAB version that has been registered as a tool named R2022b.  

```groovy
// Declarative Pipeline
pipeline {
   agent any
   tools {
       matlab 'R2022b'
   }
    stages {
        stage('Run MATLAB Command') {
            steps {
               runMATLABCommand 'myscript'
            }       
        }                
    } 
}
```

If you define your pipeline using scripted pipeline syntax, use the `tool` keyword followed by the name of the tool to retrieve the path to the MATLAB root folder. Then, prepend the MATLAB `bin` folder to the PATH environment variable.

```groovy
// Scripted Pipeline
node {
    def matlabver
    stage('Run MATLAB Command') {
        matlabver = tool 'R2022b'
        if (isUnix()) {
            env.PATH = "${matlabver}/bin:${env.PATH}"   // Linux or macOS agent
        } else {
            env.PATH = "${matlabver}\\bin;${env.PATH}"   // Windows agent
        }     
        runMATLABCommand 'myscript'
    }
}
```
You also can invoke MATLAB as a Jenkins tool when you perform a matrix build in your pipeline project. This example uses three MATLAB versions (specified in an `axis` block using their tool names) to run a set of MATLAB commands and tests. 

```groovy
// Declarative Pipeline
pipeline {
    agent any
    stages {
        stage('BuildAndTest') {
            matrix {
                agent any
                axes {
                    axis {
                        name 'MATLAB_VERSION'
                        values 'R2021b', 'R2022a', 'R2022b'
                    }
                }
                tools {
                    matlab "${MATLAB_VERSION}"
                }
                stages {
                    stage('Run MATLAB commands') {
                        steps {
                            runMATLABCommand 'ver, pwd'
                        }
                    }
                    stage('Run MATLAB Tests') {
                        steps {
                            runMATLABTests(testResultsJUnit: 'test-results/results.xml',
                                           codeCoverageCobertura: 'code-coverage/coverage.xml')
                        }
                    }
                }
            }
        }
    }
}
```

## See Also
* [Run MATLAB Tests on Jenkins Server](examples/Run-MATLAB-Tests.md)<br/>
* [Continuous Integration with MATLAB and Simulink](https://www.mathworks.com/solutions/continuous-integration.html)

---
layout: page
title: How to write MeshKit compatible errors
permalink: project/contributing/contributing-error
abstract: How to declare errors in Meshery components.
language: en
type: project
category: contributing
list: include
---

Meshery pervasively uses MeshKit as a golang and infrastructure management-specific library in all of its components. MeshKit helps populate error messages with a uniform and useful set of informative attributes.

To help with creating error codes, MeshKit contains a tool that analyzes, verifies and updates error codes in Meshery source code trees. It extracts error details into a file that can be used for publishing all error code references on the Meshery [error codes reference page](https://docs.meshery.io/reference/error-codes). The objective to create this was to avoid centralized handling of error codes and automating everything

In order to create a Meshery error object, you will need to create a custom wrapper object for the native golang error. This can be done from the <a href="https://github.com/meshery/meshkit/tree/master/errors">MeshKit Error</a> package.

This tool will create a couple of files, one of them is designed to be used to generate the error reference on the Meshery Documentation website. The file errorutil_analyze_summary.json contains a summary of the analysis, notably lists of duplicates etc.

## Conventions to follow while creating errors

1. Errors names and codes are namespaced to components, i.e. they need to be unique within a component, which is verified by this tool.
1. Errors are not to be reused across components and modules.
1. Error codes are not to be set as integer. CI will take care of updating Error codes from a string to an integer.
1. Running `make error` analyzes your code and returns any warnings to be aware of.
1. Capitalize the first letter of the every error description.
1. Using errors.NewDefault(...) is deprecated. This tool emits a warning if its use is detected.
1. Use errors.New(...) from MeshKit to create actual errors with all the details.
  This is often done in a factory function. It is important that the error code variable is used here, not a literal.
  Specify detailed descriptions, probable causes, and remedies. They need to be string literals, call expressions are ignored.
  This tool extracts this information from the code and exports it.
  For the Code argument in the errors.New use the same Error name and append a "Code" after it. e.g error name : ErrApplyManifest then the error code is ErrApplyManifestCode
1. Set the value to any string, like "replace_me" (no convention here), e.g. ErrApplyManifestCode = "replace_me".
1. By convention, error codes and the factory functions live in files called error.go. The tool checks all files, but updates only error.go files.

Use the `errors.New()` function to create a new instance of the error object and pass situation-specific attributes as function arguments. These attributes are:

- Code
- Short Description
- Long Description
- Probable Cause
- Suggested Remediation

### Syntax

{% capture code_content %}errors.New(ErrExampleCode, errors.Alert, []string{"<short-description>"}, []string{"<long-description>"}, []string{"<probable-cause>"}, []string{"<suggested remediation>"}){% endcapture %}
{% include code.html code=code_content %}
## Example

In this example we are creating an Error for being unable to marshal JSON

{% capture code_content %}var (
	// Error code
    ErrMarshalCode= "replace_me"

    //Static errors (for example)
    ErrExample = errors.New(ErrExampleCode, errors.Alert, []string{"<short-description>"}, []string{"<long-description>"}, []string{"<probable-cause>"}, []string{"<suggested remediation>"})
)

// Dynamic errors
//Error Name
func ErrMarshal(err error, obj string) error {
	return errors.New(ErrMarshalCode, errors.Alert, []string{"Unable to marshal the : ", obj}, []string{err.Error()}, []string{}, []string{})
}{% endcapture %}
{% include code.html code=code_content %}

### Replacing old Error Codes

Old

{% capture code_content %}bd, err := json.Marshal(providers)
	if err != nil {
		http.Error(w, "unable to marshal the providers", http.StatusInternalServerError)
		return
	}{% endcapture %}
{% include code.html code=code_content %}

New

{% capture code_content %}bd, err := json.Marshal(providers)
  if err != nil {
          obj := "provider"
          http.Error(w, ErrMarshal(err, obj).Error(), http.StatusInternalServerError)
          return
      }{% endcapture %}
{% include code.html code=code_content %}

## Replacing logrus

There already exists an [interface for logger](https://github.com/meshery/meshkit/blob/master/logger/logger.go) in MeshKit.<br><br>

{% include alert.html type="warning" title="WARNING" content="To enforce the use of meshkit errors, meshkit logger was designed such that it only works with meshkit errors. If a non-meshkit error is logged through the logger, it would panic and kill the process. See: <a href='https://github.com/meshery/meshkit/pull/119'>meshkit#119</a> for more insight." %}

#### Defining a Logger

{% capture code_content %}type Logger struct {
	log   logger.Handler
}{% endcapture %}
{% include code.html code=code_content %}

#### Debug

##### Old

{% capture code_content %}logrus.Debugf("meshLocationURL: %s", meshLocationURL){% endcapture %}
{% include code.html code=code_content %}

##### New

{% capture code_content %}l.log.Debug("meshLocationURL: ", meshLocationURL){% endcapture %}
{% include code.html code=code_content %}

#### Error

##### Old

{% capture code_content %}logrus.Errorf("error marshaling data: %v.", err){% endcapture %}
{% include code.html code=code_content %}

##### New

{% capture code_content %}l.log.Error(ErrMarshal(err, obj)){% endcapture %}
{% include code.html code=code_content %}

## A small program using meshkit errors and logger

{% capture code_content %}package main

import (
	"fmt"
	"os"

	meshkitErrors "github.com/meshery/meshkit/errors"
	"github.com/meshery/meshkit/logger"
	"github.com/sirupsen/logrus"
	"github.com/spf13/viper"
)

var (
	// CI will replace `test_code` with new error code
	ErrOpeningFileCode = "test_code"
)

func main() {
	logLevel := viper.GetInt("LOG_LEVEL")
	if viper.GetBool("DEBUG") {
		logLevel = int(logrus.DebugLevel)
	}
	log, err := logger.New("test", logger.Options{
		Format:   logger.SyslogLogFormat,
		LogLevel: logLevel,
	})
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	// logging meshkit error
	err = openFileWithMeshkitError("some.txt")
	if err != nil {
		log.Error(err)
	}
	// OUTPUT
	// ERRO[2021-11-10T17:31:28+05:30] open some.txt: no such file or directory
	// app=test code=1001 probable-cause="empty string passed as argument .file with this name doesn't exist"
	// severity=2 short-description="unable to open file" suggested-remediation="pass a non-empty string as
	// filename .create file before opening it"

	// logging non meshkit error
	err = openFile("some.txt")
	if err != nil {
		log.Error(err)
	}
	// OUTPUT
	// ERRO[2024-07-01T19:09:09+05:30] open some.txt: no such file or directory
	// app=test code= probable-cause= severity=0 short-description= suggested-remediation=

}

// this returns a non meshkit error
func openFile(name string) error {
	_, err := os.Open(name)
	return err
}

// this returns a meshkit error
func openFileWithMeshkitError(name string) error {
	_, err := os.Open(name)
	return ErrOpeningFile(err)
}

func ErrOpeningFile(err error) error {
	return meshkitErrors.New(ErrOpeningFileCode, meshkitErrors.Alert, []string{"unable to open file"},
		[]string{err.Error()},
		[]string{"empty string passed as argument ", "file with this name doesn't exist"},
		[]string{"pass a non-empty string as filename ", "create file before opening it"})
}{% endcapture %}
{% include code.html code=code_content %}


[![license](https://img.shields.io/badge/license-Apache%20License%202.0-blue.svg?style=flat)](http://www.apache.org/licenses/LICENSE-2.0)
[![Build Status](https://travis-ci.org/mariodavid/cuba-component-health-check.svg?branch=master)](https://travis-ci.org/mariodavid/cuba-component-health-check)

# CUBA Platform Component - Health Check

This application component automatically checks your [CUBA](https://www.cuba-platform.com/) application if it is considered "healthy".
It will show the results of the health check to the administrator of the application with information on what went wrong and information
on how to solve the issue.

You can define your own health checks at development time or at runtime that will the state of the application. Here are some examples of this checks:

* Is a dependent service alive and able to take requests
* Does the database contain a Report you created and need in the app
* Do the custom filters stored in the database fit to the entity model
* Have all database changelog scripts been applied successfully

## Installation
Currently you have to [download](https://github.com/mariodavid/cuba-component-health-check/archive/master.zip) the app-component manually and import it into Studio. After opening it in studio, you have to execute "Run > Install app component". 
After that you can go into your project and add the dependency to you project through "Project Properties > Edit > custom components (+) > cuba-component-health-check".

*Note*: This manual installation step will probably simplify with Version 6.6 of CUBA and studio.


## Health check overview

## Define custom health checks


### Development time health checks

For a lot of health checks you already know at development time what you want to check. 
These checks should be defined just as regular Java / Groovy class within your CUBA application.
 
To define custom health checks, you have to create a class that extends [DefaultHealthCheck](https://github.com/mariodavid/cuba-component-health-check/blob/master/modules/core/src/de/diedavids/cuba/healthcheck/core/healthchecks/DefaultHealthCheck.java)
in the core module of your application.


````
@Component
public class WeatherOfficeHealthCheck extends DefaultHealthCheck {


    @Inject
    WeatherOfficeService weatherOfficeService;
    
    @Override
    public HealthCheckReportDetail check() {
    
        if (weatherOfficeService.isHot()) {
            return error("get outta here, catch a drink and keep calm");
        }
        else if(weatherOfficeService.isWarm()) {
            return warning("You can work on, but make sure you are ready to party");
        }
        else {
            return success("Nothing to see here. Get a coffee...")
        }
    }

    @Override
    protected String getConfigurationCode() {
        return "weather-office-health-check-code";
    }
}
````

In the `check()` method you define the logic that should be checked. The return value is a `HealthCheckReportDetail` object that defines the outcome of the check.
To not deal with the return type directly, you can use the helper methods:


    success(String message)
    success(String message, String detailedMessage)
    warning(String message)
    warning(String message, String detailedMessage)
    error(String message)
    error(String message, String detailedMessage)


The method `getConfigurationCode()` returns the `code` of the entity [HealthCheckConfiguration](https://github.com/mariodavid/cuba-component-health-check/blob/master/modules/global/src/de/diedavids/cuba/healthcheck/entity/HealthCheckConfiguration.java) that corresponds with this health check.

*Note: Your health check class has to be a Spring bean (`@Component`) in order to get picked up by the check-runtime.*

There are a few subclasses already available for you that will make defining a health check a little easier:

* `DatabaseEntityInstanceAvailableHealthCheck` - checks for a given instance of an entity to exist in the db
* `ShellExecutionHealthCheck` - executes a shell command and verfies its outcome


### Runtime health checks

Oftentimes there is a need to define health checks at runtime. For this use case you can create a `CustomHealthCheckConfiguration` that has a script attached.
To create a runtime health check, go to `Administration > Health Check > Health Check Configurations`.

![Screenshot runtime health checks](https://github.com/mariodavid/cuba-component-health-check/blob/master/img/custom-health-check-configuration.png)


## Inital checks

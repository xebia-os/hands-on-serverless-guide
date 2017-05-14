# Getting Started with Apache OpenWhisk 

This week I started learning about Apache OpenWhisk. [Apache OpenWhisk](http://openwhisk.org/) is an open source Serverless platform. Serverless platform means a platform that enables developers to build applications that execute code in reaction to events. This new way of building applications is different from the way we are used to build applications. Serverless architecture makes you write applications that are composed of many small functions sticthed together by services provided by Serverless platform.

## OpenWhisk terminology

There are five main core concepts of OpenWhisk. These are:

1. **Feed**: They are used to configure an external event source to fire events.
2. **Trigger**: Triggers are called when events are produced by event sources. For example, when a new commit message is pushed to the Github, Github WebHook event source will invoke the trigger corresponding to it. A trigger corresponds to a class of events. 
3. **Action**: Action is a  piece of code that developers write to tell what should happen when a certain trigger is fired. Actions are not directly linked to a trigger. You write your function in your language. The only thing expected is that they should have a main function with a particular signature. An action can be triggered manually by passing the right set of parameters. This makes them easy to test in isolation.
4. **Rule**: Rule associate trigger to an action. A single action can be linked to multiple triggers making it execute on different conditions. Similarly, a single trigger can invoke multiple actions as well.
5. **Package**: Packages enable sharability of a set of related actions. This allow developers to share their actions with fellow developers and reuse stuff.

## Getting started

The easiest way to run OpenWhisk platform on your machine is to run OpenWhisk inside a virtual machine. OpenWhisk team has provided Vagrant configuration file to setup OpenWhisk platform with minimum fuss. Just download Vagrant from [official site](https://www.vagrantup.com/downloads.html). Once you have Vagrant installed, do the following:

```bash
$ git clone --depth=1 https://github.com/apache/incubator-openwhisk.git openwhisk
$ cd openwhisk/tools/vagrant
$ vagrant up
```

The above will take time depending on your internet connection and whether you have Ubuntu trusty Vagrant box on your machine or not. So, please be patient!




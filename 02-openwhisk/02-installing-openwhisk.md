# Installing Apache OpenWhisk using Vagrant

Today, I wanted to give Apache OpenWhisk a try so I decided to run on my Mac using the Vagrant setup described in the documentation. The setup mentioned in the documentation is as simple as running a couple of commands.

```Bash
$ git clone --depth=1 https://github.com/apache/incubator-openwhisk.git openwhisk
$ cd openwhisk/tools/vagrant
$ ./hello
```

> ***Note: `$` signifies prompt. You don't have to type it.***

To run above commands, you need to have vagrant and virtual box installed on your machine.

The above mentioned command will download and install all the required dependencies like Java, Kafka, Docker, Scala, Python dev tools, Ansible.

I expected that this would be smooth and I will be able to get OpenWhisk running on my machine. As it turned out, it took me more than 4 hours to get it running. The problem that I faced was that `sudo apt-get install python-dev -y` command failed. This is required to install Ansible.  Ansible is a configuration management tool that makes installing software easy and repeatable.

The `sudo apt-get install python-dev -y` command failed with the following error.

```
Reading package lists... Done
Building dependency tree      
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 python-dev : Depends: libpython-dev (= 2.7.5-5ubuntu3) but it is not going to be installed
              Depends: python2.7-dev (>= 2.7.5-1~) but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

Because `python-dev` failed to install so Ansible installation failed with `missing python.h` error.

I looked at the Vagrantfile and it was calling a script `all.sh`. This script installs multiple things. 

I got the setup working by changing the order of install scripts. I moved `ansible.sh` after the `pip.sh`. This solved the issue and installation went on smoothly. Below is the working `all.sh` file for your reference.

```Bash
#!/bin/bash
SOURCE="${BASH_SOURCE[0]}"
SCRIPTDIR="$( dirname "$SOURCE" )"

ERRORS=0


function install() {
    "$1/$2" &
    pid=$!
    wait $pid
    status=$?
    printf "$pid finished with status $status \n\n"
    if [ $status -ne 0 ]
    then
        let ERRORS=ERRORS+1
    fi
}

echo "*** installing basics"
install "$SCRIPTDIR" misc.sh

echo "*** installing python dependences"
install "$SCRIPTDIR" pip.sh

echo "*** installing ansible"
install "$SCRIPTDIR" ansible.sh

echo "*** installing java"
install "$SCRIPTDIR" java8.sh

echo "*** install scala"
install "$SCRIPTDIR" scala.sh

echo "*** installing docker"
install "$SCRIPTDIR" docker.sh

echo install all with total errors number $ERRORS
exit $ERRORS
```


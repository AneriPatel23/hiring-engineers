Your answers to the questions go here.

# Aneri_Patel_Solutions_Engineer

I am using Ubuntu VM via Vagrant. The virtual box used is Oracle VM Virtual box. I had never used vagrant before. So, it was the first time I was using. But, the references link of the datadog was very helpful. It has very specific and understandable explanation of everything, step by step. 

## Setting up the environment:

These are the commands I used to set up linux environment on vagrant:

```
 $vagrant init hashicorp/precise64

 $vagrant up

 $vagrant ssh
```

The system was up and running immediately. Every time I want to connect to the Ubuntu VM, I run ```vagrant up``` and ```vagrant ssh``` command to ssh to the VM and bring the default machine up with virtualbox provider.

Once I faced an issue connecting to VM. When I executed ```vagrant up``` command, this is what I got:

```
The guest machine entered an invalid state while waiting for it
to boot. Valid states are 'starting, running'. The machine is in the
'unknown' state. Please verify everything is configured
properly and try again.
```
I checked the state of Linux VM on Virtual box but it displayed that the system is in ```running``` state while vagrant was saying that machine is in ```unknown``` state. I googled this error. 

I came across this stackoverflow link:
[stackoverflow](https://stackoverflow.com/questions/42765576/vagrant-machine-enters-invalid-state-randomly)

I found out that it is a virtual box issue and this issue has been reported many times in that version. So, I updated the version of Virtual box to the latest stable version and executed the ```vagrant up``` command again and everything was working fine now.


**Setting up the vagrant environment:**

![vagrant_setup](https://github.com/AneriPatel23/images/blob/master/1.png)

## Collecting Metrices:

### Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

After setting up Ubuntu VM via Vagrant, I installed the datadog agent using this command:

```
DD_API_KEY=196f66f0b5580cfc5e6cfd6948bXXXXX bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"
```
You can find the easy-install command based on the agent you are using on the datadog portal. As I am using Ubuntu, I came across this command via ```Integration > Agent > Ubuntu``` section in my datadog portal. 

Once running this command, agent is installed on VM and ```datdog.yaml``` configuration file is generated in ```/etc/datadog-agent```.
I added some tags to this configuration file. This is the configuration file: 

```
api_key: 196f66f0b5580cfc5e6cfd6948bXXXXX

tags:
    database:primary
    region:east    
    role:admin
    
apm_config:
   enabled: true
 ```
Make sure you add tags with proper indentation. The tags won’t be reflected if they are not inserted at proper indentation. 

You can see the hostmap in the ```Infrastructure/HostMap``` section of datadog portal. You can also filter hosts based on the tags specified in the configuration file.

**This is the host map in datadog:**

![hostmap](https://github.com/AneriPatel23/images/blob/master/hostmap.png)

### Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

To install MySQL integration, I went to ```Integrations > MySQL``` section in datadog portal. It has all the instructions and commands, step by step on how to configure MySQL integration on datadog agent and install it. The procedure is listed below:

Firstly I installed MySQL server on VM. These are the commands to install MySQL on Ubuntu:

```
$ sudo apt-get update

$ sudo apt-get install mysql-server   //to install mysql server

$ sudo service mysql start //to start mysql service

```

To configure Mysql integration in datadog, these are the steps I followed:

-       Create a datadog user with replication rights in your MySQL server. Install Mysql server and run the following queries:
```
CREATE USER 'datadog'@'localhost' IDENTIFIED BY 'GIE14HTirTtq;oza7UhyP8gx';
GRANT REPLICATION CLIENT ON *.* TO 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;
GRANT PROCESS ON *.* TO 'datadog'@'localhost';
GRANT SELECT ON performance_schema.* TO 'datadog'@'localhost';

mysql -u datadog --password='GIE14HTirTtq;oza7UhyP8gx' -e "show status" | \
grep Uptime && echo -e "\033[0;32mMySQL user - OK\033[0m" || \
echo -e "\033[0;31mCannot connect to MySQL\033[0m"
mysql -u datadog --password='GIE14HTirTtq;oza7UhyP8gx' -e "show slave status" && \
echo -e "\033[0;32mMySQL grant - OK\033[0m" || \
echo -e "\033[0;31mMissing REPLICATION CLIENT grant\033[0m"
```
-	Configure the agent to connect to MySQL. Edit ```conf.d/mysql.yaml``` file.
```
init_config:
instances:
	server: localhost
user: datadog
pass: GIE14HTirTtq;oza7UhXXXXX
options:
	replication: 0
	galera_cluster: 1
```
-	Restart the agent.

-	Execute the ```info``` command to verify that the integration check has passed. 

**MySQL integration: (successfully installed) :**

![MySQL Integration](https://github.com/AneriPatel23/images/blob/master/3.png)

### Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

The custom metric file ```mycheck.py``` is created and shown below. The custom Agent check submits a metric named ```my_metric``` with a random value between 0 to 1000. I created the ```mycheck.py``` file in ```/etc/datadog-agent/checks.d``` and its corresponding ```mycheck.yaml``` config file in ```/etc/datadog-agent/conf.d ``` directory. The file names of both ```.py``` file and ```.yaml``` file should be same. Also make sure that you put ```.py``` check file in ```checks.d``` and ```.yaml``` file in ```conf.d``` and not anywhere else, otherwise you will encounter an error saying ``` Error: no valid check found```. 

**mycheck.py**

```
from random import randint
from checks import AgentCheck

class mycheck(AgentCheck):
    def check(self, instance):
        value=randint(0,1000)
        my_metric=self.init_config.get('my_metric',value)
        self.gauge('my_metric', '%s' %  my_metric)

if __name__=='__main__':
    check.check(instance)
    
  ```

The ```mycheck.yaml``` sets the collection interval to 45 seconds so that the metric is submitted once every 45 seconds. 

**mycheck.yaml**
```
init_config:

instances:
    [{min_collection_interval: 45}]
  ```
You can execute the check file using the following command:

For agent v5: ``` sudo -u dd-agent -- dd-agent check <check_name>```

For agent v6: ``` sudo -u dd-agent -- datadog-agent check <check_name>```

**Bonus Question Can you change the collection interval without modifying the Python check file you created?**

By changing the value of min_collection_interval parameter in the .yaml file of the custom metric, we can change the collection interval of the custom metric. By default, it is 15-20 seconds.

## Visualize data:
 
### Using the Datadog API, a timeboard is created which contains my custom metric scoped over the host.

This is the ```timeboard.py``` script file that creates a timeboard. I face some issues while creating timeboard. There was problem accessing the datadog module using the older ```pip version 1.0```. I had the error regarding the SSL connection. So, I searched on internet and found a workaround to download and install datadog module. I updated pip to its latest version, ran this ``` sudo apt-get update ``` to get the updated packages and finally installed datadog module using ```pip install datadog```.

**timeboard.py**
```
from datadog import initialize, api

options = {
    'api_key': '196f66f0b5580cfc5e6cfd6948bXXXXX',
    'app_key': '3fe788698c117720609755ca53eaca1ad8eXXXXX'
}

initialize(**options)

title = "My Timeboard"
description = "An informative timeboard."

graphs = [{
    "definition": {    
        "events": [],        
        "requests": [        
            {"q": "my_metric{*}"}
        ],        
        "viz": "timeseries"        
    },    
    "title": "Average Memory Free"    
}]
template_variables = [{
    "name": "precise64",    
    "prefix": "host",    
    "default": "host:precise64"    
}]
read_only = False

print api.Timeboard.create(title=title,
                     description=description,                     
                     graphs=graphs,                     
                     template_variables=template_variables,                     
                     read_only=read_only)
 ```

This is ```gettb.py``` script file that gets all timeboards.

**gettb.py**
```
from datadog import initialize, api

options = {
    'api_key': '196f66f0b5580cfc5e6cfd6948bXXXXX',    
    'app_key': '3fe788698c117720609755ca53eaca1ad8eXXXXX'    
}

initialize(**options)

print api.Timeboard.get_all()
```

**This is the timeboard created in datadog: **


![Timeboard](https://github.com/AneriPatel23/images/blob/master/7-timebaord_graph.png)

## Monitoring data:


### Created a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

•	Warning threshold of 500

•	Alerting threshold of 800

•	And ensure that it will notify you if there is No Data for this query over the past 10m.

Creating the monitors was straightforward. The documentation had explanation of all the variables like is_alert, is_warning, is_no_Data and more. I referred to the documentation for every step and was able to do it successfully with the help of that. I have created different messages based on system in alert, warning or no_data state. Host name and host ip is displayed by surrounding it in sets of curly braces. 

**This is the monitor created:**


![Monitor](https://github.com/AneriPatel23/images/blob/master/8-monitor%20created.png)

The monitor sends me an email whenever the monitor is triggered.

It creates different messages based on whether the monitor is in an Alert, Warning, or No Data state. 

**This is the email notification when No_data:**


![no_data](https://github.com/AneriPatel23/images/blob/master/10-nodata_email.png)

**Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:**

**I scheduled downtime that silences email alert notifications all day on Sat-Sun.**


![sat-sun_off](https://github.com/AneriPatel23/images/blob/master/12-scheduled%20downtime.png)


**I also created a recurring downtime that silences email alert notifications from 7pm to 9am daily on M-F. **


![mon-fri_off](https://github.com/AneriPatel23/images/blob/master/13-recurring%20downtime.png)


**These are the screenshots of events triggered on the monitor:**


![](https://github.com/AneriPatel23/images/blob/master/14.png)



![](https://github.com/AneriPatel23/images/blob/master/15.png)



![](https://github.com/AneriPatel23/images/blob/master/16.png)



![](https://github.com/AneriPatel23/images/blob/master/17.png)



![](https://github.com/AneriPatel23/images/blob/master/18.png)


## Collecting APM data:

**my_app.py**
```
from flask import Flask
import logging
import sys

#Have flask use stdout as the logger

main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)
app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='5050')
```
I installed pip by downloading ```get-pip.py``` and executing it using following commands. 
```
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

$ python get-pip.py
```
After the latest version of pip was installed,

-	I executed the following command to install ddtrace:
```pip install ddtrace```

-	I created python file ```my_app.py``` :

-	Executed the following command to collect traces of python app.
```ddtrace-tun python my_app.py```

Initially when I ran ```ddtrace-run``` command, I was getting this error:

```
INFO:werkzeug: * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
2018-09-17 03:16:44,430 - werkzeug - INFO -  * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
ERROR:ddtrace.writer:cannot send services to localhost:8126: [Errno 111] Connection refused
2018-09-17 03:16:45,424 - ddtrace.writer - ERROR - cannot send services to localhost:8126: [Errno 111] Connection refused
```

I checked logs file located at ```/var/log/datadog/trace-agent.log``` and found that it was throwing error at some line number in ```datadog.yaml``` file. I resolved the error and executed the ```ddtrace-run``` command again. Now there was no error and it shows the following output:


![](https://github.com/AneriPatel23/images/blob/master/20.png)


But, I am still not getting any traces reported. I even tried by manually inserting middleware by adding the code for middleware in the app file. But I still have no traces collected. 

I executed ```my_app.py``` file and also tried printing something on the console to make sure the file is running. It worked well but while executing the file with ```ddtrace-run```, I don’t get any traces being reported back to the datadog.

The logs file also does not show any error. 

**Is there anything creative you would use Datadog for?**

I can think of using Datadog for Parking spots. I live in California and getting a parking spot is one head scratching task. This would be a very useful application of Datadog as the usage of parking lots is tremendous in some area and moderate in some areas. It would be very useful to get information of which parking spots are being used the most, at what hours is the usage maximum, what hours is the usage minimum, which area need more number of parking spots, etc. Based on that information, one can analyze the graph between the demand and availability of parking spots. 

**Can you change the collection interval without modifying the Python check file you created?**
    
Yes, we can change the collection interval without modifying python check file by setting min_collection_interval value in the   corresponding yaml file created in conf.d. 
```
init_config:

 instances: 
    [{min_collection_interval: 45}]
 ``` 
 **What is the Anomaly graph displaying?**
 
 Anomaly graph displaying allows you to detect changes in behaviour of metric by comparing it to its past behaviour. It displays if there is any sudden change in the behaviour of metric and alerts if there is any anomaly.
 
 There is an anomalies function in the Datadog query language. When you apply this function to a series, it returns the usual results along with an expected “normal” range.
 
**What is the difference between a Service and a Resource?**

 A service is a set of processes that do the same job. Services can be one of these types: web, database, cache, custom. For instance, a simple web application may consist of two services: a single webapp service and a single database service while a more complex environment can be modularized into more services.
      
 A Resource is a specific action for a service. For an instance, for SQL database, a resource can be a simple query statement.
 
**Link to datadog:**

https://app.datadoghq.com/infrastructure/map?host=575361155&fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&groupby=availability-zone&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=green_to_orange&paletteflip=false&node_type=host&app=(no-namespace)


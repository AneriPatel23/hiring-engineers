Your answers to the questions go here.

Aneri_Patel_Solutions_Engineer

I am using Ubuntu VM via Vagrant. The virtual box used is Oracle VM Virtual box. I had never used vagrant before. So, it was the first time I was using. But, the references link of the datadog was very helpful. It has very specific and understandable explanation of everything, step by step. 

Setting up the environment:

These are the commands I used to set up linux environment on vagrant:
$vagrant init hashicorp/precise64
$vagrant up
$vagrant ssh

The system was up and running immediately. 
Image link of setting up the vagrant environment: https://drive.google.com/open?id=1U5VJb6_B2j0NOKwZJ_0p3fmcF4u6f-Pi
Collecting Metrices:
Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.
After setting up Ubuntu VM via Vagrant, I installed the datadog agent using this command:


DD_API_KEY=196f66f0b5580cfc5e6cfd6948bcd913 bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"


Datadog.yaml was generated. I added some tags to this configuration file. This is the configuration file: 

api_key: 196f66f0b5580cfc5e6cfd6948bcd913
tags:
   - database:primary
   - region:east
   - role:admin
apm_config:
   enabled: true

This is the host map in datadog: https://drive.google.com/open?id=1mXcHGh2y3WpPjIM0FoGdIN0KTGoJ0jc-

Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
I configured MySQL integration and installed it in datadog. 

To configure Mysql integration in datadog, these are the steps I followed:

•	Create a datadog user with replication rights in your MySQL server. Install Mysql server and run the following queries:

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

•	Configure the agent to connect to MySQL. Edit conf.d/mysql.yaml file.
init_config:
instances:
-	server: localhost
user: datadog
pass: GIE14HTirTtq;oza7UhyP8gx
tags:
-	optional_tag1
-	optional_tag2
options:
-	replication: 0
-	galera_cluster: 1

•	Restart the agent.

•	Execute the “info” command to verify that the integration check has passed. 

Image link to MySQL integration: (successfully installed) https://drive.google.com/open?id=1qHJygQQEmfavWZrX0596jgsufCMX_G1h

Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.
The custom metric file mycheck.py is created and shown below. The custom Agent check submits a metric named my_metric with a random value between 0 to 1000. I created the mycheck.py file in checks.d directory in datadog-agent and its corresponding mycheck.yaml config file in conf.d directory.

mycheck.py
from random import randint
from checks import AgentCheck

class mycheck(AgentCheck):
    def check(self, instance):
        value=randint(0,1000)
        my_metric=self.init_config.get('my_metric',value)
        self.gauge('my_metric', '%s' %  my_metric)

if __name__=='__main__':
    check.check(instance)

The mycheck.yaml sets the collection interval to 45 seconds so that the metric is submitted once every 45 seconds. 

mycheck.yaml
init_config:

instances:
    [{min_collection_interval: 45}]

Bonus Question Can you change the collection interval without modifying the Python check file you created?
By changing the value of min_collection_interval parameter in the .yaml file of the custom metric, we can change the collection interval of the custom metric. By default, it is 15-20 seconds.
Visualize data:
 
Using the Datadog API, a timeboard is created which contains my custom metric scoped over the host.

This is the timeboard.py script file that creates a timeboard. I face some issues while creating timeboard. There was problem accessing the datadog module from the https link. I had the error regarding the SSL connection. So, I found a workaround to download and install datadog module. I updated pip to its latest version, ran this  “ sudo apt-get update ” to get the updated packages and finally installed datadog module using “ pip install datadog”.

timeboard.py
from datadog import initialize, api

options = {
    'api_key': '196f66f0b5580cfc5e6cfd6948bcd913',
    'app_key': '3fe788698c117720609755ca53eaca1ad8e9ad14'
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

This is gettb.py script file that gets all timeboards.

gettb.py
from datadog import initialize, api

options = {
    'api_key': '196f66f0b5580cfc5e6cfd6948bcd913',
    'app_key': '3fe788698c117720609755ca53eaca1ad8e9ad14'
}

initialize(**options)

print api.Timeboard.get_all()

Image link of the timeboard created: 
https://drive.google.com/open?id=10kuJmGFW9v7HIpj6iRgG8vytz9UHFCuC

Monitoring data:

Monitor created:
Created a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:
•	Warning threshold of 500
•	Alerting threshold of 800
•	And ensure that it will notify you if there is No Data for this query over the past 10m.
Creating the monitors was straightforward. The documentation had explanation of all the variables like is_alert, is_warning, is_no_Data and more. I referred to the documentation for every step and was able to do it successfully with the help of that. I have created different messages based on system in alert, warning or no_data state. Host name and host ip is displayed by surrounding it in sets of curly braces. 
Image link of the monitor created: https://drive.google.com/open?id=1XmnBQ9V35XJCg4iw7_w2ihQqmbgupcEJ
The monitor sends me an email whenever the monitor is triggered.
It creates different messages based on whether the monitor is in an Alert, Warning, or No Data state. 
Image link of email notification when No_data: https://drive.google.com/open?id=1ITu9yKPsFs-1_QAOZLFO3BamA12oXMkp
Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

I scheduled downtime that silences email alert notifications all day on Sat-Sun.

Image link to that is here: https://drive.google.com/open?id=1hAdJ5D6S77Q3efoIwPrTx3ajlpgUZyBs

I also created a recurring downtime that silences email alert notifications from 7pm to 9am daily on M-F. 

Image link to that is here:
https://drive.google.com/open?id=1VlKn2EKzWEnIexpASAcweXoGJC-HeEO7

Here are links to screenshots of events triggered on the monitor:

https://drive.google.com/open?id=1N-2656ex2uyxeuu4pl-t-AJNuSxr2h_g
https://drive.google.com/open?id=1DRyUYRnqy-jIEaWKChE8pFllIh0e8Z1T
https://drive.google.com/open?id=1w50cUwy4madDSKgDbARXamNdeky-GqnA
https://drive.google.com/open?id=11OvIyN8aM3YVjJKxHyBvXQ1iV3SEpb_n
https://drive.google.com/open?id=1jn0o7vML8ljcfjA0xbb0AHbjrrTJgLlR

Collecting APM data:

my_app.py
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
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

I installed pip by downloading get-pip.py and executing it using following commands. 
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python get-pip.py

After the latest version of pip was installed,

•	I executed the following command to install ddtrace:
pip install ddtrace

•	I created python file my_app.py :

•	Executed the following command to collect traces of python app.
ddtrace-tun python my_app.py

Initially when I ran ddtrace-run command, I was getting this error:

INFO:werkzeug: * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
2018-09-17 03:16:44,430 - werkzeug - INFO -  * Running on http://0.0.0.0:5050/ (Press CTRL+C to quit)
ERROR:ddtrace.writer:cannot send services to localhost:8126: [Errno 111] Connection refused
2018-09-17 03:16:45,424 - ddtrace.writer - ERROR - cannot send services to localhost:8126: [Errno 111] Connection refused

I checked logs file located at /var/log/datadog/trace-agent.log and found that it was throwing error at some line number in datadog.yaml file. I resolved the error and executed the ddtrace-run command again. Now there was no error and it shows the following output:

https://drive.google.com/open?id=1Ml0kf_Z4SVmuFAIwH-XCkGiQbap0uYYl

But, I am still not getting any traces reported. I even tried by manually inserting middleware by adding the code for middleware in the app file. But I still have no traces collected. 

I executed my_app.py file and also tried printing something on the console to make sure the file is running. It worked well but while executing the file with ddtrace-run, I don’t get any traces being reported back to the datadog.

The logs file also does not show any error. 

Is there anything creative you would use Datadog for?

I can think of using Datadog for Parking spots. I live in California and getting a parking spot is one head scratching task. This would be a very useful application of Datadog as the usage of parking lots is tremendous in some area and moderate in some areas. It would be very useful to get information of which parking spots are being used the most, at what hours is the usage maximum, what hours is the usage minimum, which area need more number of parking spots, etc. Based on that information, one can analyze the graph between the demand and availability of parking spots. 

Link to datadog:

https://app.datadoghq.com/infrastructure/map?host=575361155&fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&groupby=availability-zone&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=green_to_orange&paletteflip=false&node_type=host&app=(no-namespace)


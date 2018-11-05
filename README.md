## Metricbeat


Coralogix provides integration with Metricbeat using Logstash.

### Table of contents

1. Prerequisites
2. Usage
3. Installation
4. Configuration

### Prerequisites
Have Logstash installed, for more information on how to install: https://www.elastic.co/guide/en/logstash/current/installing-logstash.html  

Have Metricbeat installed, for more information on how to install: https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html

### Usage

**Private Key** – A unique ID which represents your company, this Id will be sent to your mail once you register to Coralogix.

**Application Name** – Used to separate your environment, e.g. SuperApp-test/SuperApp-prod.

**SubSystem Name** – Your application probably has multiple subsystems, for example: Backend servers, Middleware, Frontend servers etc. in order to help you examine the data you need, inserting the subsystem parameter is vital.

### Installation

```bash
logstash-plugin install logstash-output-coralogix_logger
```

If you are not sure where logstash-plugin is located, you can check here:  
https://www.elastic.co/guide/en/logstash/current/dir-layout.html

### Configuration

Open your Logstash configuration file and add Beats input and Coralogix output. (More information about Beats Input plugin: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-beats.html)

```javascript
input {
  beats {
    port => 5044
    codec => "json"
  }
}

output {
if [metricset][module] == "system" {
# Replace "system" with any module name.

    coralogix_logger { 
        config_params => {
            "PRIVATE_KEY" => "11111111-1111-1111-1111-111111111111"
            "APP_NAME" => "Logstash Tester"
            "SUB_SYSTEM" => "Logstash subsystem"
        } 
        log_key_name => "message"
        timestamp_key_name => "YOUR_TIMESTAMP_FIELD"

        log_key_name => "system"
        # Replace "system" with any module name.

        is_json => true
    }
  }
}  
```
**Input**  
Port number is mandatory. 

**Output**  
The first key (config_params) is mandatory while the other two are optional.

**Timestamp:**  Coralogix automatically generates the timestamp based on the log arrival time.  If you rather use your own timestamp, use the “timestamp_key_name” to specify your timestamp field, and it will be read from your log. 

```json
{
    "@timestamp": "2017 - 04 - 03 T18: 44: 28.591 Z",
    "@version": "1",
   "process": {
      "fd": {
        "limit": {
          "hard": 1048576,
          "soft": 1048576
        },
        "open": 22
      },
      "cpu": {
        "start_time": "2018-10-28T05:34:15.000Z",
        "total": {
          "norm": {
            "pct": 0
          },
          "pct": 0,
          "value": 310
        }
      },
      "ppid": 1,
      "pgid": 539,
      "cwd": "/var/cache/bind",
      "pid": 539,
      "cmdline": "/usr/sbin/named -f -u bind",
      "state": "sleeping",
      "name": "named"
      }
}
```
Because your Metricbeat events are converted to JSON object, in the case you don’t want to send the entire JSON, rather just a portion of it, you can write the value of the key you want to send in the log_key_name.

For instance, in the above example, if you write log_key_name "x_edge_location" then only the value of "x_edge_location" key will be sent to Coralogix. If you do want to send the entire message then you can just delete this key.

Restart Logstash.  

Open your Metricbeat configuration file. (More information about Metricbeat:  https://www.elastic.co/guide/en/beats/metricbeat/master/index.html)

```yaml
output.logstash:
  hosts: ["localhost:5044"]
```

Restart Metricbeat. 

# logtash-employement-test
Create a Logstash configuration file allowing for ingestion of the given Security Log Data via multiple “Logstash Input Plugin”, then parsing the data as JSON formatted data containing required fields.

## Background
This is a demostration on how to use logstash with plugins to parse the security log listed below:
><14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname="Virus Found" computername="contosopc42" computerip="216.58.194.142" severity="1"

Per instruction from the potential employer, the output should be JSON formatted data containing (but not exclusively) the following fields::
- description: "Virus Found" 
- hostname: "contosopc42" 
- source_ip: "216.58.194.142" 
- severity: "High"

## Method
In order to achieve the required output, the following tools and Logstash plugins are needed:

**Online tools**
- Grok debugger tool: https://www.javainuse.com/grok
- Grok Constructor: https://grokconstructor.appspot.com/do/constructionstep
- Logstash Grok Pattern: https://github.com/elastic/logstash/blob/v1.4.0/patterns/grok-patterns

**Logstash plugins**
- Grok
- Mutate

## Instruction:
1. Open Grok Constructor and select "Incremental Construction"
2. Copy and Paste the log to the blank box and then click "Go!"
3. Following the step-by-step guidence and seperate each field with provided field template, and you should get the pattern similar to below:
>%{SYSLOGPROG} %{TIMESTAMP_ISO8601} %{HOSTNAME} %{WORD} %{NUMBER} - - %{EMAILLOCALPART}%{QUOTEDSTRING} %{EMAILLOCALPART}%{QS} %{EMAILLOCALPART}%{QUOTEDSTRING} %{EMAILLOCALPART}%{QUOTEDSTRING}
4. Copy and Paste the generated pattern to Grok debugger tool, alone with the log.
5. Check each field and cross-reference with Logstash Grok Pattern till you can see the desired output format from the page, and your Grok Pattern should be similar to below:
>%{NOTSPACE:session_id}\s%{TIMESTAMP_ISO8601:timestamp}\s%{WORD:main_host}\s%{WORD:process}\s%{NUMBER:session}\s%{NOTSPACE}\s%{NOTSPACE}\s%{NOTSPACE}%{QS:description}\s%{NOTSPACE}%{QS:hostname}\s%{NOTSPACE}%{QS:ip}\s%{NOTSPACE}%{NOTSPACE}%{NUMBER:severity}%{NOTSPACE}
6. Open Visual Studio Code or similar text editor, and enter the basic template for Logstash configuration file:
>input {stdin{}}

>output {stdout{codec => rubydebug}}

7. To use the Grok filter, we enter below between "input" and "output":
>filter {
    grok{match => {"message" => }
}
8. Then copy and paste the working Grok Pattern from the Grok Debugger to the right of "message" =>, and make sure to add "" on both end, similar to below:
>grok{match => {"message" => "%{NOTSPACE:session_id}\s%{TIMESTAMP_ISO8601:timestamp}\s%{WORD:main_host}\s%{WORD:process}\s%{NUMBER:session}\s%{NOTSPACE}\s%{NOTSPACE}\s%{NOTSPACE}%{QS:description}\s%{NOTSPACE}%{QS:hostname}\s%{NOTSPACE}%{QS:source_ip}\s%{NOTSPACE}%{NOTSPACE}%{NUMBER:severity}%{NOTSPACE}"}}

9. Since the desired output request to show ***severity: "High"***, an IF statement is required to convert targeted integrer to string, showing "high" or "low". Therefore, we create IF statement listed below:
>if [severity] > '1' {mutate { replace => { "severity" => "low" } } }

>else if [severity] == '1' {mutate { replace => { "severity" => "High" } }} 

>else {mutate { replace => { "severity" => "Medium" } }}
10. Now we combine the Grok and IF statement together, then the config setting should similar to below:
>input {stdin{}}

>filter {
    grok{match => {"message" => "%{NOTSPACE:session_id}\s%{TIMESTAMP_ISO8601:timestamp}\s%{WORD:main_host}\s%{WORD:process}\s%{NUMBER:session}\s%{NOTSPACE}\s%{NOTSPACE}\s%{NOTSPACE}%{QS:description}\s%{NOTSPACE}%{QS:hostname}\s%{NOTSPACE}%{QS:source_ip}\s%{NOTSPACE}%{NOTSPACE}%{NUMBER:severity}%{NOTSPACE}"}}
    if [severity] > '1' {mutate { replace => { "severity" => "low" } } } 
    else if [severity] == '1' {mutate { replace => { "severity" => "High" } }} 
    else {mutate { replace => { "severity" => "Medium" } }}
}

>output {stdout{codec => rubydebug}}

11. Open terminal on your computer, then run Logstash:
>[directory where logstash is located]/bin/logstash -f [directory where the .conf file is located]/yourfile.conf

12. Once Logstash is started, copy and paste the log to the terminal, the returned result should be similar to below:
>"event" => {
        "original" => "<14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname=\"Virus Found\" computername=\"contosopc42\" computerip=\"216.58.194.142\" severity=\"1\""
    },
       "severity" => "High",
        "message" => "<14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname=\"Virus Found\" computername=\"contosopc42\" computerip=\"216.58.194.142\" severity=\"1\"",
      "timestamp" => "2016-12-25T09:03:52.754646-06:00",
        "process" => "antivirus",
    "description" => "\"Virus Found\"",
           "host" => {
        "hostname" => "your computer name"
    },
        "session" => "2496",
      "main_host" => "contosohost1",
     "session_id" => "<14>1",
       "hostname" => "\"contosopc42\"",
       "@version" => "1",
     "@timestamp" => 2022-09-09T19:46:27.075474Z,
             "ip" => "\"216.58.194.142\""

13. In order to remove the extra ***\"***, we will need to add a Mutate filter to the config file, as listed below:
>filter {
    mutate { gsub => ["description","\"",""]}
    mutate { gsub => ["hostname","\"",""]}
    mutate { gsub => ["source_ip","\"",""]}
}

14. Run Logstash again and with both filter, the final result should be similar to below:
>     "session_id" => "<14>1",
          "event" => {
        "original" => "<14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname=\"Virus Found\" computername=\"contosopc42\" computerip=\"216.58.194.142\" severity=\"1\""
    },
        "message" => "<14>1 2016-12-25T09:03:52.754646-06:00 contosohost1 antivirus 2496 - - alertname=\"Virus Found\" computername=\"contosopc42\" computerip=\"216.58.194.142\" severity=\"1\"",
      "timestamp" => "2016-12-25T09:03:52.754646-06:00",
     "@timestamp" => 2022-09-09T20:13:27.002278Z,
    "description" => "Virus Found",
       "@version" => "1",
           "host" => {
        "hostname" => "your computer name"
    },
       "hostname" => "contosopc42",
        "session" => "2496",
      "source_ip" => "216.58.194.142",
      "main_host" => "contosohost1",
        "process" => "antivirus",
       "severity" => "High"

## Challenge
1. It is a very delicated procedure to setup Logstash for the first time, and the program will **NOT** run unless the config file is %100 correct, so it will take hours or days till you can get into the program.
2. In order to correctly identify each field with Grok pattern, please rely on the Grok Debugger to find the correct pattern before running anything in the terminal.
3. Pay attention to the required output, in case we need to convert certain field from one type to another, in this case, the severity level need to be "high", instead of "1".
4. Always make a clean output as much as possible, with help from Mutate filter.

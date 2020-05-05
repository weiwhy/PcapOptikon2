# PcapOptikon2
PCAPOptikon is a project that will provide an easy way to analyze pcap using the latest version of Suricata and Zeek.
It can also save Suricata and Zeek logs in Elasticsearch using the new Elasticsearch Common Schema or the original field names.

# Install & Uninstall
Install Docker-CE and docker-compose:
- https://docs.docker.com/install/linux/docker-ce/ubuntu/
- https://docs.docker.com/compose/install/

Start everything
```
sudo docker-compose up -d
```

Use the following command to check if everything is working properly:
```
sudo docker-compose ps
Name                       Command                       State                    Ports          
---------------------------------------------------------------------------------------------------------
pcapopt_elasticsearch   /usr/local/bin/docker-entr ...   Up (health: starting)   9200/tcp, 9300/tcp      
pcapopt_filebeat        /usr/local/bin/docker-entr ...   Up                                              
pcapopt_kibana          /usr/local/bin/dumb-init - ...   Up (health: starting)   127.0.0.1:5601->5601/tcp
pcapopt_suricata        suricata --runmode=single  ...   Exit 0                                          
pcapopt_zeek            zeek -C -r /pcap/test.pcap ...   Exit 0
```

Suricata and Zeek are ran only when analythig a pcap. It's correct their state is in `Exit 0`.

To stop everything run:
```
sudo docker-compose stop
```

To delete all containers run
```
sudo docker-compose down
```

# Usage

## Managing suricata Signatures
Use the following command to download all ET rules:
```
sudo docker-compose run --entrypoint='suricata-update -f' suricata
```
This command will download the rules and create a rule file in `./config/suricata/rules/suricata.rules`.

If you want to test custom rules add them in `./config/suricata/rules/custom.rules` file.

## Analyzing a PCAP
Put a file named `test.pcap` inside the `pcap` folder. It's mandatory to call the file `test.pcap` because the filename is hardcoded in `docker-compose.yaml` (I know this sucks, if you know how to fix this a pull request is gladly accepted)

Start zeek and suricata containers:
```
sudo docker-compose up zeek suricata
```

The containers will print the output on the console and exit when they finish processing the pcap.
You can see the results on Kibana: http://localhost:5601

## Zeek Extracted Files

The file extraction plugin is automatically loaded in Zeek and the extracted files can be found in `./zeek/extracted_files`. The plugin configuration is in this file `./config/zeek/site/file-extraction/config.zeek`. You can add additional filetypes in the following file:

Example: to add Microsoft Office file extraction add the following line to `./config/zeek/site/file-extraction/config.zeek`:
```
@load ./plugins/extract-ms-office.zeek
```
You can find all supported file types in `.config/zeek/site/file-extraction/plugins`.

# Advanced Usage

## Light weight usage: ditching elasticsearch (the hacker way)
If you prefer using the command line (because [reasons](https://giphy.com/gifs/YQitE4YNQNahy/html5)) you can find suricata and zeek logs in `./logs` directory.

If you are gpippi don't waste time starting filebeat/elasticsearch/kibana go to `./zeek/site/local.zeek` and comment out line 3. Then start analyzing a new pcap and enjoy plaintext, tab separated zeek logs. `awk` all the way, baby!

Even if you'd like to use directly the log file I suggest to keep them in `.json` format and use `jq` utility to query them. You can read a pretty good `jq` primer [here](https://www.gibiansky.com/blog/command-line/jq-primer/index.html)

## Fucking elastic common schema, give me normal logs!
If you don't like the remapping of Suricata and Zeek logs in ECS (elastic common schema) you can revert back to original field names.

To do so you need to comment filebeat entry on `docker-compose.yaml` and comment out logstash entry. Now zeek and suricata logs are stored in `zeek` and `suricata` elasticsearch index keeping the original field name. You need to add the index-pattern manually in kibana.
[TODO screenshots]

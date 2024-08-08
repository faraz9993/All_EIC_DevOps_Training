# Day 21:
### In this task, I have created a few shelll scripts, ran prometheus on the docker container, download and added node exporter in the prometheus configuration file, ran a few PromQL commands.

### Shell scripts to perform basic system operations, such as checking disk usage, memory usage, and CPU load.
```
#!/bin/bash

echo "
 
-------------------------------------------" >> output.txt

top -b -n 1 | head -n 35 >> output.txt
```


###  A shell script to automate log management tasks such as log rotation and archiving. This script should include the ability to compress old logs and delete logs older than a specified number of days.

```
#!/bin/bash

LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/var/log/myapp/archive"
DAYS_TO_KEEP=30                

sudo mkdir -p /var/log/myapp
sudo mkdir -p /var/log/myapp/archive
sudo mkdir -p "$ARCHIVE_DIR"

timestamp=$(date +"%Y%m%d%H%M%S")
for log_file in "$LOG_DIR"/*.log; do
    if [ -f "$log_file" ]; then
        mv "$log_file" "$ARCHIVE_DIR/$(basename "$log_file" .log)_$timestamp.log"
    fi
done

find "$ARCHIVE_DIR" -type f -name "*.log" -exec gzip "{}" \;

find "$ARCHIVE_DIR" -type f -name "*.gz" -mtime +$DAYS_TO_KEEP -exec rm -f "{}" \;

echo "Script executed successfully."
```
### Refactoring of the previous scripts to include loops, conditionals and functions for modularity. Implement error handling to manage potential issues during script execution.

```
#!/bin/bash


LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/var/log/myapp/archive"
DAYS_TO_KEEP=30

handle_error() {
    local error_message="$1"
    echo "Error: $error_message" >&2
    exit 1
}
create_directory() {
    local dir="$1"
    if [ ! -d "$dir" ]; then
        mkdir -p "$dir" || handle_error "Failed to create directory $dir. Check permissions."
    fi
}
rotate_logs() {
    local log_dir="$1"
    local archive_dir="$2"
    local timestamp=$(date +"%Y%m%d%H%M%S")

    for log_file in "$log_dir"/*.log; do
        if [ -f "$log_file" ]; then
            local new_log_file="$archive_dir/$(basename "$log_file" .log)_$timestamp.log"
            mv "$log_file" "$new_log_file" || handle_error "Failed to move $log_file to $new_log_file. Check permissions."
        else
            echo "No log files found in $log_dir to rotate."
        fi
    done
}
compress_logs() {
    local archive_dir="$1"

    for log_file in "$archive_dir"/*.log; do
        if [ -f "$log_file" ]; then
            gzip "$log_file" || handle_error "Failed to compress $log_file. Check permissions."
        fi
    done
}
delete_old_logs() {
    local archive_dir="$1"
    local days_to_keep="$2"

    find "$archive_dir" -type f -name "*.gz" -mtime +$days_to_keep -exec rm -f "{}" \; || handle_error "Failed to delete old logs in $archive_dir. Check permissions."
}
main() {
    if [ ! -d "$LOG_DIR" ]; then
        handle_error "Log directory $LOG_DIR does not exist."
    fi

    create_directory "$ARCHIVE_DIR"
    rotate_logs "$LOG_DIR" "$ARCHIVE_DIR"
    compress_logs "$ARCHIVE_DIR"
    delete_old_logs "$ARCHIVE_DIR" "$DAYS_TO_KEEP"
    echo "Log rotation and management tasks completed successfully."
}
main
echo "Script executed successfully."
```
### Below is a script that reads through system and application logs, identifies common issues (e.g., out of memory, failed service starts) and provides troubleshooting steps based on log analysis.


```
#!/bin/bash

SYSTEM_LOG="/var/log/syslog"
APPLICATION_LOG="/var/log/myapp/application.log"

declare -A common_issues
common_issues=(
    ["out_of_memory"]="Possible Root Cause: System ran out of memory. Consider adding more RAM or optimizing memory usage of applications."
    ["failed_service_start"]="Possible Root Cause: Service failed to start. Check service configuration and dependencies."
    ["disk_space"]="Possible Root Cause: Insufficient disk space. Free up space or add more storage."
    ["permission_denied"]="Possible Root Cause: Permission denied. Check file permissions and user privileges."
    ["segmentation_fault"]="Possible Root Cause: Segmentation fault. Investigate application code for bugs or memory issues."
)


analyze_logs() {
    local log_file="$1"
    echo "Analyzing $log_file..."

    while IFS= read -r line; do
        for issue in "${!common_issues[@]}"; do
            if [[ "$line" =~ $issue ]]; then
                echo "Issue Found: $issue"
                echo "Log Entry: $line"
                echo "${common_issues[$issue]}"
                echo ""
            fi
        done
    done < "$log_file"
}

if [ ! -f "$SYSTEM_LOG" ]; then
    echo "Error: System log file $SYSTEM_LOG not found."
    exit 1
fi

if [ ! -f "$APPLICATION_LOG" ]; then
    echo "Error: Application log file $APPLICATION_LOG not found."
    exit 1
fi

analyze_logs "$SYSTEM_LOG"
analyze_logs "$APPLICATION_LOG"

echo "Script executed successfully."
```

### Installation of Prometheus:
### I have used docker container for using Prometheus. I pulled an image named prom/prometheus and ran the container using it with the below command:

```
docker run -p 9090:9090 -itd prom/prometheus
```

### Set-up Node Exporter:
### For setting-up the Node exporter, I used an ec2 instance. I SSH into ec2 instance and ran a docker container with prometheous prom/node-exporter image. Then, I went into the prometheus configuration file using below command.
```
docker exec -it compassionate_swirles /bin/sh
```

### I made the below changes in the /etc/prometheus/prometheus.yml file:

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
                    
  - job_name: "node"
    static_configs:                 
      - targets: ["http://3.149.27.16:9100/"]
```

### Below image shows that both the targets are up and running.

### I have also created a series of PromQL queries to monitor system performance, such as CPU usage, memory usage, and disk I/O.

```
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

rate(node_disk_read_bytes_total[5m])

rate(node_disk_read_bytes_total[5m])

(node_filesystem_size_bytes{fstype!~"tmpfs|overlay"} - node_filesystem_free_bytes{fstype!~"tmpfs|overlay"}) / node_filesystem_size_bytes{fstype!~"tmpfs|overlay"} * 100

rate(node_network_receive_bytes_total{device!="lo"}[5m])
```
### Below are the images showing the graphs of all the above shown commands.
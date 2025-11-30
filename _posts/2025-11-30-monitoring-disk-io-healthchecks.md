---
layout: post
title: "Monitoring Disk I/O Health with Healthchecks.io"
date: 2025-11-30
categories: [monitoring]
tags: [bash, cron, monitoring]
---

Healthchecks.io is a simple yet powerful service for monitoring scheduled tasks and cron jobs. Instead of actively checking your systems, it works on a "dead man's switch" principle: your scripts ping the service regularly, and if a ping is missed or fails, you receive an alert. In this guide, I'll show you how to set up automated disk I/O monitoring using a bash script and Healthchecks.io.

## How Healthchecks.io Works

The concept is straightforward:
1. Create a check on healthchecks.io and receive a unique URL
2. Your script performs its task
3. On success, the script pings the URL
4. On failure, the script pings the URL with a `/fail` suffix
5. If no ping arrives within the expected schedule, you get notified

This approach is perfect for monitoring cron jobs, backups, or any scheduled maintenance tasks.

## Creating a Health Check

First, sign up for a free account at [healthchecks.io](https://healthchecks.io). The free tier is generous enough for personal use and small projects, it allows you to create up to 20 checks.

After logging in:
1. Click "Add Check"
2. Give your check a descriptive name (e.g., "Disk I/O Monitor")
3. Set the schedule to match your cron job
    * We'll use hourly in this example, use the "cron" mode for period and simply paste the same schedule that you will add to the cronjob: `0 * * * *`
    * Set the timezone of the check to match your server time
4. Set the grace period (this is up to you, how fast you want to be informed about potential issues)
4. Configure notification channels (email, Slack, Discord, etc.)
5. Save the check

You'll receive a unique ping URL that looks like: `https://hc-ping.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

## Example: Disk I/O Monitoring Script

Let's create a practical monitoring script that tests disk I/O operations and reports the status to Healthchecks.io. This script will:
- Test read and write operations on mounted filesystems
- Perform both text and binary I/O tests
- Check kernel logs for I/O errors
- Report success or failure to Healthchecks.io

Create the monitoring script:

```bash
#!/usr/bin/bash
set -euo pipefail

# Healthchecks.io URLs
HC_PING_OK="https://hc-ping.com/your-unique-id-here"
HC_PING_FAIL="https://hc-ping.com/your-unique-id-here/fail"

# Define mountpoints to test
MOUNTPOINTS=(
    "/mnt/disk1"
    "/mnt/disk2"
)

log_file="/var/log/io_monitor.log"
touch "$log_file"

has_error=0

for mp in "${MOUNTPOINTS[@]}"; do
    if mountpoint -q "$mp"; then
        echo "[$(date)] Testing $mp..." | tee -a "$log_file"

        testfile="$mp/iotest.txt"
        testfile_bin="$mp/iotest.bin"
        expected="Test from $(hostname) at $(date +%s)"

        # Text write test
        if ! echo "$expected" > "$testfile"; then
            echo "[$(date)] Text write failed on $mp" | tee -a "$log_file"
            has_error=1
            continue
        fi

        sync

        # Text read and verify
        actual=$(cat "$testfile" 2>/dev/null || echo "")
        if [[ "$actual" != "$expected" ]]; then
            echo "[$(date)] Text read/verification failed on $mp" | tee -a "$log_file"
            has_error=1
        fi

        sleep 5
        rm -f "$testfile"

        # Binary write test
        if ! /bin/dd if=/dev/urandom of="$testfile_bin" bs=1M count=1 oflag=direct status=none; then
            echo "[$(date)] Binary write failed on $mp" | tee -a "$log_file"
            has_error=1
        fi

        # Binary read test
        if ! /bin/dd if="$testfile_bin" of=/dev/null bs=1M iflag=direct status=none; then
            echo "[$(date)] Binary read failed on $mp" | tee -a "$log_file"
            has_error=1
        fi

        sleep 5
        rm -f "$testfile_bin"

        # Check kernel logs for I/O errors
        if dmesg | tail -n 50 | grep -qi "I/O error"; then
            echo "[$(date)] I/O error detected in kernel logs during $mp test" | tee -a "$log_file"
            has_error=1
        fi
    else
        echo "[$(date)] $mp is not mounted, skipping." | tee -a "$log_file"
    fi
done

# Report to healthchecks.io
if [ "$has_error" -eq 0 ]; then
    curl -fsS -m 10 --retry 3 -o /dev/null "$HC_PING_OK"
else
    curl -fsS -m 10 --retry 3 -o /dev/null "$HC_PING_FAIL"
fi
```

Save this script (e.g., as `/root/io_monitor.sh`) and make it executable:

```bash
chmod +x /root/io_monitor.sh
```

## Scheduling with Cron

Add the script to your crontab to run hourly:

```bash
crontab -e
```

Add this line:

```
0 * * * * /root/io_monitor.sh
```

This configuration will test your disks every hour and automatically notify you through your configured channels if any issues are detected.

## Understanding the Script

The script performs several types of tests:

1. **Mountpoint verification**: Ensures the filesystem is actually mounted before testing
2. **Text I/O test**: Writes a timestamped string and verifies it can be read back correctly
3. **Binary I/O test**: Uses `dd` with direct I/O to test larger block operations
4. **Kernel log inspection**: Checks for recent I/O errors in system logs
5. **Status reporting**: Pings Healthchecks.io with success or failure status

The `set -euo pipefail` at the beginning ensures the script fails fast on errors, and the error tracking mechanism (`has_error` variable) allows all mountpoints to be tested before reporting the final status.

## Why This Approach Works

This monitoring strategy offers several advantages:

- **Proactive detection**: Issues are caught before they cause major problems
- **Simple notification**: No need to configure complex monitoring infrastructure
- **Flexible alerting**: Healthchecks.io supports multiple notification channels like email, slack etc...
- **Minimal overhead**: The script only runs periodically and uses minimal resources
- **Easy maintenance**: All logic is contained in a single, readable bash script

The dead man's switch principle means you'll be notified not only when tests fail, but also if the script stops running entirely (due to system failure, cron misconfiguration, etc.).

## Conclusion

Healthchecks.io provides an elegant solution for monitoring scheduled tasks without requiring complex monitoring infrastructure. By combining it with simple bash scripts, you can create robust monitoring for critical system operations like disk I/O, backups, or any other scheduled maintenance tasks. The service's free tier is suitable for most personal and small business needs, making it an excellent choice for reliable, low-maintenance monitoring.

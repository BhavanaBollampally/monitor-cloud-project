# MonitorCloud — System Resource Monitor With Cloud Log Storage

MonitorCloud is a small learning project focused on Bash scripting, log management, Docker containerization, simple CI/CD, and basic cloud integration. The script checks system resources, writes timestamped logs, removes older logs locally, and can upload logs to AWS S3 when the tools and credentials are available. A cron job on an EC2 instance runs the container in the background to simulate continuous monitoring.

The goal behind this project was to practice how different DevOps components connect in a real workflow.

## Features

- Checks CPU, memory, and disk usage
- Calculates WARN and CRITICAL based on thresholds
- Creates timestamped log files in a dedicated folder
- Deletes logs older than a retention period
- Uploads log files to AWS S3 when AWS CLI is detected
- Containerized using Docker for portability
- Automatically built and tagged using GitHub Actions
- Run continuously via cron on an EC2 instance
- AWS S3 lifecycle rule removes older cloud logs automatically

## Architecture Overview

Local Bash script → Docker container → GitHub Actions builds new images → DockerHub → EC2 cron job pulls the latest image → Script uploads logs to AWS S3.

AWS S3 stores logs in year/month/day folder structure. A lifecycle rule is used to expire objects after a defined number of days.

## Technology Used

- Bash
- Ubuntu Linux
- Docker
- GitHub Actions
- AWS S3
- IAM roles and policies
- Cron
- Linux utilities (`grep`, `awk`, `df`, `free`, `find`)

## How It Works

1. `monitor.sh` checks CPU, memory, and disk using `/proc` and native commands.
2. Each check returns an exit code indicating OK, WARN, or CRITICAL.
3. A final status line is built using all three exit codes.
4. The script writes a log entry with a timestamp.
5. Logs older than the retention limit are removed using `find`.
6. If AWS CLI is installed, the script uploads the log to S3 inside a date-based folder.
7. Docker provides a consistent runtime for all environments.
8. GitHub Actions builds and pushes updated images automatically.
9. Cron on EC2 pulls the `latest` image and runs the script on a schedule.

## Repository Folder Structure

MonitorCloud/
├── monitor.sh
├── log_monitor/
├── Dockerfile_monitorcloud
└── README.md


CI/CD workflow files are stored under:

.github/workflows/


## Running Locally (No Cloud Upload)

cd MonitorCloud
chmod +x monitor.sh
./monitor.sh


Logs will appear in:

MonitorCloud/log_monitor/


## Running With Docker Locally

Build the image:

docker build -t monitorcloud:v1 -f Dockerfile_monitorcloud .


Run:

docker run monitorcloud:v1


Logs will appear inside `/app/log_monitor` inside the container.

## Continuous Operation on EC2 (Cron)

The following cron entry runs every two minutes:

*/2 * * * * docker pull bhavanabollampally/monitorcloud:latest && docker run --rm bhavanabollampally/monitorcloud:latest >> /home/ubuntu/cron_monitor.log 2>&1


This pulls the updated image automatically whenever GitHub Actions publishes a new version.

## CI/CD Pipeline (GitHub Actions)

On relevant pushes:

- Code is checked out
- Docker is set up
- A new image is built and tagged
- Tags include a version number and `latest`
- Image is pushed to DockerHub

This models a basic container delivery pipeline.

## Cloud Storage Details (AWS S3)

- Logs are uploaded using AWS CLI
- IAM policy restricts the S3 actions
- IAM instance profile is used (instead of hardcoded credentials)
- Logs are stored under `YYYY/MM/DD`
- A lifecycle rule deletes objects automatically after seven days

## Why I Built This

I wanted to understand how simple monitoring scripts evolve into real workflows, and how Docker, CI/CD, cloud storage, and scheduled jobs can work together. This project helped me learn debugging across different environments and reinforced best practices around cloud credentials.

## Possible Future Improvements

- Send notifications on CRITICAL status
- Export metrics to CloudWatch
- Visual dashboarding using Grafana
- Deploy using Terraform end-to-end
- Run inside Kubernetes for scheduling

---

Built as part of my DevOps learning journey.







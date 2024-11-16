# Leapwork Run Reports

`leapwork_run_reports` is an application that integrates with Leapwork’s automation suite, enabling automated report generation for completed runs. It listens for Leapwork scheduler notifications, processes relevant automation data, and generates PDF reports with associated screenshots. This application is optimized for deployment in Docker and has customizable settings through environment variables.

## Features

### 1. **Report Generation for Automation Runs**
   - Generates detailed PDF reports for completed automation runs.
   - Includes key information such as run details, keyframes, status, and execution timestamps.
   - Filters keyframes based on customizable criteria (e.g., specific markers like `"@@"` in `BlockTitle`).

### 2. **Screenshots and Directory Structure**
   - Captures and stores screenshots for each run in structured directories based on `flow_id` and `run_id`.
   - Screenshots are saved within a `screenshots` subdirectory for each unique `run_id`.
   - Screenshots are also embedded in the generated PDF report, maintaining aspect ratio.

### 3. **Flexible API Configuration**
   - Supports configurable API endpoints for fetching automation run details, keyframes, screenshots, and related data.
   - Uses environment variables to configure URLs and access keys for easy deployment across different environments.

### 4. **FastAPI Integration**
   - Provides a FastAPI interface with endpoints to handle scheduling requests, view payloads, and generate reports.
   - Supports cross-origin requests with configurable CORS settings.

### 5. **Detailed Logging and Debugging**
   - Configurable logging with options to output logs to console and files.
   - Supports setting log levels, formats, and file locations through environment variables.
   - Includes structured log formatting for easy monitoring and troubleshooting.

---

## How It Works

1. **Scheduler Notification**:
   - The application listens for notifications from the Leapwork scheduler. Once an automation run is completed, it sends the details to the `/api/process-schedule` endpoint.

2. **Payload Parsing**:
   - The incoming payload is parsed to extract `ScheduleId`, `RunItems`, `FlowId`, and other relevant details.
   - The application identifies the most recent run and processes only that run’s data to generate a PDF report.

3. **Report Generation**:
   - A PDF report is generated based on `flow_id` and `run_id`, including key details of the automation run, filtered keyframes, and screenshots.
   - The report is saved in a structured directory under `export/{flow_id}/{run_id}`.

4. **Flexible Configuration**:
   - The application is fully configurable through environment variables, making it ideal for deployment in Docker containers.
   - The base URL for Leapwork APIs and various server settings can be customized as required.

---

## Configuration

The application uses environment variables for easy customization. The following variables are supported:

| Environment Variable                             | Description                                                                 | Default Value                       |
|--------------------------------------------------|-----------------------------------------------------------------------------|-------------------------------------|
| `LEAPWORK_RUN_REPORTS_LEAPWORK_URL`              | Base URL of the Leapwork server for API requests                            | `http://leapwork201.lux4rd0.com:9001`   |
| `LEAPWORK_RUN_REPORTS_ACCESS_KEY`                | Access key for authenticating with the Leapwork API                         | `He2P2sKq7sbWjaBu`                  |
| `LEAPWORK_RUN_REPORTS_FASTAPI_SERVER_HOST`       | Host address for the FastAPI server                                         | `0.0.0.0`                           |
| `LEAPWORK_RUN_REPORTS_FASTAPI_SERVER_PORT`       | Port for the FastAPI server                                                 | `6080`                              |
| `LEAPWORK_RUN_REPORTS_ALLOWED_ORIGINS`           | CORS allowed origins for FastAPI requests                                   | `*`                                 |
| `LEAPWORK_RUN_REPORTS_LOG_DIR`                   | Directory for storing log files                                             | `/app/logs`                         |
| `LEAPWORK_RUN_REPORTS_LOG_FILE_ENABLED`          | Enables logging to a file (`true` or `false`)                               | `False`                             |
| `LEAPWORK_RUN_REPORTS_LOG_LEVEL`                 | Logging level (e.g., `DEBUG`, `INFO`, `WARNING`)                            | `DEBUG`                             |
| `LEAPWORK_RUN_REPORTS_LOG_FORMAT`                | Format for log messages                                                     | `%(asctime)s %(levelname)s %(message)s` |

---

## API Endpoints

### `/api/generate/job/{job_id}`

- **Description**: Generates a report for a specified job.
- **Method**: `POST`
- **Parameters**: 
  - `job_id`: Unique ID of the job.
  - JSON body with options such as `include_pdf` and `template_flag`.
- **Response**: JSON message confirming report generation status.

### `/api/generate/flow/{flow_id}`

- **Description**: Generates a report for a specific flow.
- **Method**: `POST`
- **Parameters**: 
  - `flow_id`: Unique ID of the flow.
  - JSON body with options for `include_pdf` and `template_flag`.
- **Response**: JSON message confirming report generation status.

### `/api/process-schedule`

- **Description**: Processes a payload sent by the Leapwork scheduler, triggering report generation for each run item.
- **Method**: `POST`
- **Parameters**: None (reads from JSON payload).
- **Response**: JSON message confirming schedule processing status.

### `/api/info`

- **Description**: Captures and logs all incoming request data (headers, payload) for debugging.
- **Method**: `POST`
- **Response**: JSON message with received payload details.

---

## Directory Structure

Generated reports and screenshots are stored under the `export` directory in a structured format:

```
export/
└── {flow_id}/
    └── {run_id}/
        ├── report_{run_id}.pdf            # PDF report file for the run
        └── screenshots/
            ├── screenshot_{screenshot_id}.png   # Screenshot for the run item
            └── ...
```

---


Here’s the updated **"Deployment with Docker"** section for the documentation, incorporating your provided example `compose.yaml` file and updating the access key to a placeholder for security.

---

## Deployment with Docker

To deploy the **Leapwork Run Reports** application using Docker, you can use the following `docker-compose.yml` file:

```yaml
name: leapwork_run_reports
services:
  leapwork_run_reports_backend:
    command:
      - python
      - -m
      - backend.main
    container_name: leapwork_run_reports_backend
    environment:
      # Leapwork Configuration
      LEAPWORK_RUN_REPORTS_ACCESS_KEY: YOUR_LEAPWORK_ACCESS_KEY_HERE
      LEAPWORK_RUN_REPORTS_LEAPWORK_URL: http://leapwork201.lux4rd0.com:9001

      # FastAPI Server Configuration
      LEAPWORK_RUN_REPORTS_FASTAPI_SERVER_HOST: 0.0.0.0
      LEAPWORK_RUN_REPORTS_FASTAPI_SERVER_PORT: "6080"

      # Logging Configuration
      LEAPWORK_RUN_REPORTS_LOG_DIR: /app/logs
      LEAPWORK_RUN_REPORTS_LOG_FILE_ENABLED: "true"
      LEAPWORK_RUN_REPORTS_LOG_LEVEL: DEBUG

      # Time Zone
      TZ: America/Chicago

    image: lux4rd0/leapwork_run_reports:2024.11.2
    networks:
      default: null
    ports:
      - mode: ingress
        target: 6080
        published: "6080"
        protocol: tcp
    restart: always
    volumes:
      - /mnt/docker/leapwork_run_reports/export:/app/export
networks:
  default:
    name: leapwork_run_reports_default
```

---

### Key Points for Deployment

1. **Environment Variables**:
   - Replace `YOUR_LEAPWORK_ACCESS_KEY_HERE` with your actual Leapwork API access key.
   - Configure `LEAPWORK_RUN_REPORTS_LEAPWORK_URL` to point to your Leapwork server.

2. **Volume Mapping**:
   - The volume mapping (`/mnt/docker/leapwork_run_reports/export:/app/export`) ensures that generated reports and screenshots are persisted on the host machine.

3. **Network Configuration**:
   - Uses a dedicated Docker network named `leapwork_run_reports_default`.

4. **Time Zone**:
   - Ensures the container operates in the desired time zone (`America/Chicago`).

---

This configuration provides a secure, organized, and scalable deployment setup for the `leapwork_run_reports` application. Make sure to replace placeholders with actual values and adjust paths as necessary for your environment.

---

## Conclusion

The **`leapwork_run_reports`** application provides a streamlined solution for automating and generating detailed reports for Leapwork automation runs. With customizable API settings, structured file storage, and flexible configuration options, it’s ideal for environments that require automated report generation and real-time scheduler integration.

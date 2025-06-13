# Connecting to the running Databases

```
docker run -it --rm --network host mysql:latest mysql -h 127.0.0.1 -P 3306 -u root -p
docker run -it --rm --network host postgres:latest psql -h 127.0.0.1 -p 5432 -U postgres -d exam_db
```

# Exam Practice Databases

This project sets up MySQL and PostgreSQL databases using Docker Compose for data management exam practice. The databases are pre-configured with a `students` table for practice queries. This setup uses Docker to run client connections, eliminating the need to install MySQL or PostgreSQL clients on your system. Sensitive credentials are managed via a `.env` file.

## Prerequisites

- **Docker** and **Docker Compose** installed on your Ubuntu system.

## Project Structure

- `docker-compose.yml`: Defines MySQL and PostgreSQL services.
- `init-mysql.sql`: Initializes MySQL with a `students` table.
- `init-postgres.sql`: Initializes PostgreSQL with a `students` table.
- `.env`: Stores sensitive credentials (not committed to Git).
- `.env.example`: Template for the `.env` file.
- `mysql-data/`: Stores MySQL data (excluded from Git).
- `postgres-data/`: Stores PostgreSQL data (excluded from Git).
- `.gitignore`: Excludes `mysql-data/`, `postgres-data/`, and `.env` from version control.

## Setup

1. Clone the repository (if applicable):
   ```bash
   git clone <your-repo-url>
   cd <project-directory>
   ```

2. Create a `.env` file based on `.env.example`:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` to set your desired credentials, e.g.:
   ```
   MYSQL_ROOT_PASSWORD=password
   MYSQL_USER=exam_user
   MYSQL_PASSWORD=exam_pass
   POSTGRES_PASSWORD=password
   ```

3. Create the data directories if they don’t exist:
   ```bash
   mkdir -p mysql-data postgres-data
   ```

## Starting the Services

1. Start the databases in detached mode:
   ```bash
   docker-compose up -d
   ```

2. Verify the containers are running:
   ```bash
   docker-compose ps
   ```
   Ensure `mysql-exam` and `sql-exam` are in the `Up` state.

## Connecting to the Databases

You can connect to the databases using Dockerized `mysql` and `psql` clients, which run in temporary containers and require no local installation.

### MySQL
- **Host**: `127.0.0.1`
- **Port**: `3306`
- **Database**: `exam_db`

#### Using the Dockerized MySQL Client
- Root user:
  ```bash
  docker run -it --rm --network host mysql:latest mysql -h 127.0.0.1 -P 3306 -u root -p
  ```
  Enter the password defined in `.env` as `MYSQL_ROOT_PASSWORD`.
- Non-root user:
  ```bash
  docker run -it --rm --network host mysql:latest mysql -h 127.0.0.1 -P 3306 -u ${MYSQL_USER} -p
  ```
  Enter the password defined in `.env` as `MYSQL_PASSWORD`.

#### Verify Data
```sql
USE exam_db;
SHOW TABLES;
SELECT * FROM students;
```

### PostgreSQL
- **Host**: `127.0.0.1`
- **Port**: `5432`
- **Database**: `exam_db`
- **User**: `postgres`

#### Using the Dockerized psql Client
```bash
docker run -it --rm --network host postgres:latest psql -h 127.0.0.1 -p 5432 -U postgres -d exam_db
```
Enter the password defined in `.env` as `POSTGRES_PASSWORD`.

#### Verify Data
```sql
\dt
SELECT * FROM students;
```

## Cleaning Up After Each Session

To stop the containers and preserve data for the next session:
```bash
docker-compose down
```

To reset the databases (clear all data and start fresh):
1. Stop and remove containers:
   ```bash
   docker-compose down
   ```
2. Remove the data directories:
   ```bash
   rm -rf mysql-data postgres-data
   ```
3. Recreate the directories:
   ```bash
   mkdir mysql-data postgres-data
   ```
4. Restart the services:
   ```bash
   docker-compose up -d
   ```

## Notes
- The `students` table is initialized by `init-mysql.sql` and `init-postgres.sql` when the containers start with empty data directories.
- Data in `mysql-data/` and `postgres-data/` persists between sessions unless explicitly removed.
- The `.env` file contains sensitive credentials and should not be committed to Git. It is excluded via `.gitignore`.
- For backups, export database dumps:
  - MySQL: `docker exec mysql-exam mysqldump -u root -p${MYSQL_ROOT_PASSWORD} exam_db > backup-mysql.sql`
  - PostgreSQL: `docker exec sql-exam pg_dump -U postgres exam_db > backup-postgres.sql`
- To troubleshoot, view container logs:
  ```bash
  docker logs mysql-exam
  docker logs sql-exam
  ```
- The `--network host` flag ensures the client containers can access the database ports on the host. If you encounter connection issues, verify that ports `3306` and `5432` are not blocked or in use by other services.
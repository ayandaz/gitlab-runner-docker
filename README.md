# MySQL + phpMyAdmin Docker Setup

## What this is

A ready-to-run MySQL 8 + phpMyAdmin setup using Docker Compose.

## Usage

1. Copy `.env.example` to `.env` and edit your secrets:

```bash
cp .env.example .env

2. Start the containers:
docker-compose up -d

3. Access phpMyAdmin:
Go to http://localhost:8080 and log in using the credentials from your .env



Ports
MySQL: localhost:3306
phpMyAdmin: localhost:8080
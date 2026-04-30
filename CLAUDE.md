# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Carsties-Net10 is a microservices-based car auction platform built on .NET 10. Currently contains a single microservice (**AuctionService**) with more planned. The solution uses the newer `.slnx` format (`Carsties-Net10.slnx`).

## Build & Run Commands

```bash
# Build the AuctionService
dotnet build src/AuctionService

# Run the AuctionService (HTTP on localhost:7001)
dotnet run --project src/AuctionService

# Add a new EF Core migration
dotnet ef migrations add <MigrationName> -p src/AuctionService

# Apply migrations manually
dotnet ef database update -p src/AuctionService
```

## Infrastructure

- **PostgreSQL** runs via Docker on **host port 1010** (mapped to container port 5432) — port 5432 is intentionally avoided due to another local PostgreSQL instance.
- Start infrastructure: `docker compose up -d`
- Connection string is in `src/AuctionService/appsettings.Development.json`
- Database is auto-migrated and seeded on startup via `DbInitializer.InitDb()` in `Program.cs`

## Architecture

### AuctionService (`src/AuctionService/`)

| Layer | Location | Purpose |
|-------|----------|---------|
| Entities | `Entities/` | Domain models: `Auction` (has one `Item`), `Status` enum |
| Data | `Data/` | `AuctionDbContext` (EF Core), `DbInitializer` (migration + seed data) |
| DTOs | `DTOs/` | `AuctionDto` (response, flattens Auction+Item), `CreateAuctionDto`, `UpdateAuctionDto` |
| Mapping | `RequestHelpers/MappingProfiles.cs` | AutoMapper profiles — uses `IncludeMembers` to flatten Item into AuctionDto |
| Controllers | `Controllers/` | REST endpoints (not yet implemented) |

### Key Patterns

- **AutoMapper 16.x registration** requires a config action as the first parameter: `AddAutoMapper(_ => { }, typeof(Program).Assembly)`. The older `AddAutoMapper(assemblies)` overload no longer exists.
- **DTO flattening**: `AuctionDto` combines fields from both `Auction` and its child `Item` entity into a single flat response object using AutoMapper's `IncludeMembers`.
- **One-to-one relationship**: Each `Auction` has exactly one `Item` (the car being auctioned), with cascade delete configured.

## MCP Servers

- **context7** is configured in `.mcp.json` for querying up-to-date library documentation.

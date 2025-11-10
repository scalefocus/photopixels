# Event Sourcing Implementation
Photo Pixels application implements Event Sourcing using [Marten](https://martendb.io/) as the persistence layer for events

## Stream concept
- The application uses streams as the fundamental unit for organizing events. Each stream has a unique `streamId` that identifies a specific entity or collection.
- Streams maintain version tracking (`version` field) to track event sequence.

## Event Storage:
When Marten is configured for event sourcing, it creates several key tables: `mt_streams`, `mt_events`, `mt_event_progression`. Events are associated with specific streams through the **stream_id** foreign key

### Events
- Events are stored in `mt_events` table using Marten's event store functionality. Some of the most noticeable columns, important for Photopixels from this table are:
  - `seq_id`: Sequence number of the event
  - `stream_id`: References the stream it belongs to
  - `version`: Version number within the stream
  - `data`: JSON field containing the actual event data
  - `type`: Type of the event
  - `timestamp`: When the event occurred


### Streams
- The streams are stored in `mt_streams` table.

## Other DB Tables
Based on its configuration, Marten can automatically creates tables that starts with 'mt_doc_' (and then the entity name). If you need to add new table, you should create the entity class and then add it to the  Marten configuration block in `photopixels-backend-net\src\SF.PhotoPixels.Infrastructure\DependencyInjection.cs`.
 - The tables created here contains the `data` field with the event information

## Migrations
Marten migrations are SQL scripts that define the database schema and can be executed in sequence to evolve the database structure.
- In order to start migration when a database change is needed, read `photopixels-backend-net\docs\Migrations.md` file

## Repositories
There are two main repositories that are implemented to add events data to the stream:

### IObjectRepository
Adds events when there is na event related with modifications of images, videos and albums (crud operations, adding and removing from trash and etc.)
 - **It uses the current `userId` for streamId**

### IAlbumRepository
Adds events when there is an event related with concrete album (adding and removing photos to album and ect)
- **It uses the `albumId` for streamId**

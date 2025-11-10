# Synchronization Protocol

## Purpose
The synchronization protocol enables consistent and efficient data exchange between the server and multiple client devices — including desktop, web, and mobile applications.
Its primary goal is to ensure that each client stays in sync with the latest user data (albums, photos, and metadata) even in scenarios with no internet connectivity.

## Concept
Each user can create albums and upload photos across multiple devices.
Since devices may operate offline for extended periods, the server maintains a versioned event log of all data changes.
When a client reconnects, it can request all updates that occurred since its last known revision.

## Synchronization Flow

1. Initial Sync
- When a client logs in for the first time, it performs a full synchronization.
- The client receives the entire dataset (albums, media objects, etc.) and a current revision number (e.g., 520).
- This revision acts as a snapshot marker for future incremental updates.

2. Offline Operation
- Clients may continue to create, update, or delete content locally while offline.
- Each operation is stored in a local queue awaiting synchronization.

3. Incremental Sync
- Upon reconnection, the client sends its last known revision number to the server:
```
GET /revision/{lastKnownRevision}
```
- The server compares it with the latest revision and returns all events that occurred since that point:
```json
{
  "version": 529,
  "albums": {
    "added": {
      "2db9bef3-f981-4a45-be4e-5922a30fdfd3": 1761748754196
    },
    "updated": {},
    "deleted": []
  },
  "added": {
    "image_123": 1761748710499
  },
  "trashed": {},
  "deleted": []
}
```
- The response contains only the delta — what was added, updated, trashed, or deleted since the provided revision.

4. If a client has made local changes to the same entities, it must resolve conflicts according to its internal rules (e.g., last-write-wins or merge strategy).

## Endpoints
```
GET /revision/{revisionId}
```
Returns all state changes that have occurred since the given revision.
- `added`: Newly created media objects.
- `trashed`: Media objects moved to trash.
- `deleted`: Permanently deleted items.
- `albums.added`: Newly created albums.
- `albums.updated`: Modified albums.
- `albums.deleted`: Removed albums.

### Versioning

Each server event increments a global revision counter.

Revisions are sequential and monotonic — ensuring reliable ordering of events.


For example:
```
400 → user creates Album A
405 → user uploads Photo X
420 → user deletes Photo X
520 → user updates Album A title
```
If a client’s last sync was at revision 400, requesting /revision/400, the server will return the changes up to revision 520 (but not not return every event individually)

##### Basic Sequential Example
 Revision |   	Event Type     | 	Description            |
|---------|--------------------|---------------------------|
 400	  | MediaObjectCreated | User uploads a new photo. |
 401	  | MediaObjectUpdated | User renames the photo.   |

When the client requests `/revision/400`,
the server already knows that the photo was created and then updated —
so it only needs to report it as Added, including its final metadata.
```json
{
  "added": {
    "photo_123": 1761748710499
  },
  "updated": {}
}
```

##### Transient (Created → Trashed → Restored)
 Revision |   	Event Type              | 	Description   |
|---------|-----------------------------|-----------------|
 410	  | MediaObjectCreated          | Photo uploaded. |
 420	  | MediaObjectTrashed          | Moved to trash. |
 430	  | MediaObjectRemovedFromTrash | Restored.       |

When requesting /revision/410, the resulting state after reduction is:
- The photo exists and is active,
- The intermediate “trashed” state is irrelevant.

```json
{
  "added": {
    "photo_456": 1761748754196
  },
  "trashed": {},
  "deleted": []
}
```

#### Created → Trashed → Deleted

 Revision |   	Event Type     | 	Description      |
|---------|--------------------|---------------------|
 500	  | MediaObjectCreated | Photo uploaded      |
 510	  | MediaObjectTrashed | Moved to trash      |
 520	  | MediaObjectDeleted | Permanently deleted |

When the client syncs from revision 490 to 520, the photo never needs to appear in “added” or “trashed” collections, because its final state is deleted.

```json
{
  "added": {},
  "trashed": {},
  "deleted": ["photo_789"]
}
```
The protocol collapses **intermediate states**, ensuring the client doesn’t waste resources processing transient transitions that are no longer relevant.

#### Album object
Each album is having its own stream with the id of the album. All operations that are happening inside the album are under this stream. Right now the only supported operations are: object_to_album_created and object_to_album_deleted.

Here the rule that during synchronization is returned only the delta is kept again.

#### Image is created and then deleted

```json
{
  "id": "7945d258-c138-4879-aac5-7b451fb8330d",
  "version": 2,
  "added": {},
  "removed": []
}
```
The image is not returned, because the client doesn't need to know about it (it has been deleted) in the same interval

#### Image is created and then deleted

```json
{
  "id": "7945d258-c138-4879-aac5-7b451fb8330d",
  "version": 2,
  "added": {},
  "removed": []
}
```

#### Image has been uploaded
```json
{
  "id": "7945d258-c138-4879-aac5-7b451fb8330d",
  "version": 1,
  "added": {
    "LxupogrmzYtwlMITV3T4WPiNX_M": 1762509265291
  },
  "removed": []
}
```

#### Image has been removed from the album
```json
{
  "id": "7945d258-c138-4879-aac5-7b451fb8330d",
  "version": 1,
  "added": {},
  "removed": ["LxupogrmzYtwlMITV3T4WPiNX_M"]
}
```

#### Remove image from an album
```json
{
  "id": "7945d258-c138-4879-aac5-7b451fb8330d",
  "version": 1,
  "added": {},
  "removed": ["LxupogrmzYtwlMITV3T4WPiNX_M"]
}
```

#### Deleting an image
When an image is deleted, no matter from which source (from main gallery, from the current album, from another album), there are multiple events that have been raised:
- **One event in the main user stream** which indicates that the images is deleted
- **An event per album** in which the image belongs to, to indicate that this image is no longer available in the album

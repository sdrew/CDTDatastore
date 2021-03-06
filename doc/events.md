# Events

CDTDatastore uses NSNotificationCenter to raise events for changes in the
system.

## Threading

Events may be fired from any thread. If you're modifying the UI in response
to events (e.g., reloading a UITableView), switch to the main thread as
you'd usually do, for example using GCD:

```objc
dispatch_async(dispatch_get_main_queue(), ^{
    ...
});
```

## Database changed - CDTDatastoreChangeNotification

Defined in:

- `CDTDatastore.h`, included in `CloudantSync.h`.

Registration example:

```objc
CDTDatastore *datastore = [...];
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(dbChanged:)
                                             name:CDTDatastoreChangeNotification
                                           object:datastore];
```

Object can be `nil` or a CDTDatastore instance. In the latter case, only
notifications for that datastore will be delivered to your listener.

Fired when:

- When a document in a datastore is modified (created, updated or deleted).

In the `userInfo` dictionary for creating, updating and deleting revisions:

- `rev`: the new revision of the document.
- `winner`: the current winning revision for the document.

For changes as a result of replication, a field is added to the `userInfo`
dictionary:

- `source`: NSURL of remote database, if added due to replication.

For the `-deleteDocumentById:error:` method, which may delete multiple
revisions, in addition to the delete notifications for each revision there
is an addition notification. Its `userInfo` dictionary contains:

- `deletedRevs`: an array containing all the deleted revisions.

Listener example:

```objc
/**
 Notified that a document has been created/updated/deleted.

 This method acts on changes to documents with the ID `self.documentToWatch`.
 */
- (void) dbChanged: (NSNotification*)n {
    CDTDocumentRevision* rev = (n.userInfo)[@"rev"];

    NSString* docID = rev.docId;
    if (![docID isEqualToString:self.documentToWatch])
        return;

    if (rev.deleted) {
        // do something
    }

    NSDictionary *body = [rev documentAsDictionary];

    // Process the current document's content using the
    // body dictionary...
}
```

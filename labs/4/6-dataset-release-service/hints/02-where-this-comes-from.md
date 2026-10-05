> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [S3 multipart upload overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
  — "After you initiate a multipart upload, there is no expiry; you must
  explicitly complete or stop the multipart upload." The parts are stored and
  billed until one of those happens, and a reader of the key sees nothing.
  The same page says: "Do not use the result of this listing when sending a
  complete multipart upload request. Instead, maintain your own list of the
  part numbers that you specified when uploading parts and the corresponding
  ETag values". That sentence is the design decision this lab is built on.
- [S3 conditional writes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/conditional-writes.html)
  — "Conditional writes do not consider any in-progress multipart uploads
  requests since those are not yet fully written objects."
- [S3 consistency model](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
  — "Updates are key-based. There is no way to make atomic updates across
  keys."
- [S3 object upload limits](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html)
  — "With a single PUT operation, you can upload a single object up to 5 GB
  in size."
- [S3 object copy limits](https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html)
  — "You can create a copy of an object up to 5 GB in a single atomic
  operation."
- [OpenStreetMap planet diffs](https://wiki.openstreetmap.org/wiki/Planet.osm/diffs)
  — "Under no circumstances should you attempt to just fetch diffs by
  incrementing the sequence number as incomplete diffs may be present beyond
  the one identified in the state file." The domain's own answer to the
  problem, and the reason the pointer exists.

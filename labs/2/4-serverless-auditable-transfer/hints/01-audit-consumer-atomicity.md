> Spoilers. Open only when stuck.

# Why the audit consumer cannot assume it sees an atomic write as a unit

Read the transaction page in the DynamoDB documentation cited by this lab
beside the streams page, and hold the two against each other.

The write is atomic: the items in one transaction either all apply or none do.
The stream is not the write. It reports item changes, and a consumer reads
them as separate records, in its own time, without a marker saying which of
them arrived together. So the audit consumer can observe the first half of an
atomic write and nothing else for as long as it takes the next record to
arrive.

The design question that follows: what does the consumer publish during that
window, and what has to be carried in the records themselves for it to know
that a group is incomplete rather than finished?

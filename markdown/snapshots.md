## Snapshots

Note: OpenStack makes use of RBD snapshots all over the place. Some
examples:


### Ephemeral storage

Note:
As mentioned before, if we’re running on Nova with ephemeral storage
on RBD, *and* we’re also running on Glance on RBD, *and* we have the
Glance v2 API enabled with `show_image_direct_url = true`, then Nova
will always clone from the image snapshot. That means that immediately
when we upload an image into Glance, RBD creates a snapshot for us,
and it is from that single snapshot that instances are cloned. So
that’s just 1 snapshot per image, no more.


### Nova snapshots

Note:
When we create an instance snapshot from Nova, what RBD will do is
create a snapshot image, then create a clone from that snapshot, then
(by default) _flatten_ the clone so the snapshot can be managed
independently of its VM (if we disable this, then we save space, but
we can’t ever delete an instance that still has snapshots). 


### Cinder snapshots

Note:
Cinder volume snapshots also, obviously, map to RBD snapshots whenever
Cinder is RBD-backed.


### Cinder Backup

Note:
When we’re doing differential backups with `cinder-backup`, that’s
another area where RBD snapshots are used heavily.


## You should know:
There is more to RBD snapshots than meets the eye.

Note:
Snapshots are a tricky business in RBD. They look simple, but in
reality they’re quite complex. Now, I’m not going into the details
here too deeply, because there’s an excellent video on this from Ceph
developer Greg Farnum from the Boston summit:


## You should watch:
Ceph Snapshots for Fun and Profit (Greg Farnum)

https://youtu.be/rY0OWtllkn8

Note:
In particular: you should know and understand that snapshots aren’t
free, and that snapshots are particularly affected by one internal
detail of their implementation that causes really, really bad side
effects in the event that you

* *create* a lot of RBD snapshots,
* and subsequently, *delete many* of them, but not all.

And in an OpenStack environment, that is of course tricky, because you
don’t have control over how many, for example, Cinder snapshots or
backups your users take and subsequently throw away.

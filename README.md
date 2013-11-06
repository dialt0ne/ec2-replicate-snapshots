## ec2-replicate-snapshots

### Overview

`ec2-replicate-snapshots` has been designed to work with
[ec2-consistent-snapshot](https://github.com/alestic/ec2-consistent-snapshot) and
[ec2-expire-snapshots](https://github.com/alestic/ec2-expire-snapshots)
to copy EBS snapshot backups to another region for disaster recovery purposes.

### Usage

```
usage: ec2-replicate-snapshots [-h] [-D] [-B] -s SOURCE -d DESTINATION

Copies EC2 snapshots from one region to another

optional arguments:
  -h, --help            show this help message and exit
  -D, --debug           output at debug level
  -B, --botodebug       enable boto debugging (not enabled with -D)
  -s SOURCE, --source SOURCE
                        Source EC2 region
  -d DESTINATION, --destination DESTINATION
                        Destination EC2 region
```

### Example

```
VOLID="vol-5calable"
ec2-consistent-snapshot \
    --region us-east-1 \
    --freeze-filesystem /data \
    --description "important data" \
    $VOLID > >(
        ec2-replicate-snapshots \
            --source us-east-1 \
            --destination us-west-2
    ) 2> >( logger -t snapshot-backups )
ec2-expire-snapshots \
    --keep-first-hourly 48 --keep-first-daily 14 --keep-first-weekly 8 --keep-first-monthly 24 \
    --region us-east-1 \
    $VOLID
ec2-expire-snapshots \
    --keep-first-hourly 48 --keep-first-daily 14 --keep-first-weekly 8 --keep-first-monthly 24 \
    --region us-west-2 \
    --volume-id-in-tag MasterVolumeId \
    $VOLID
```

The above does the following:

* calls `ec2-consistent-snapshot` to do the following
    + creates a snapshot of the EBS volume `vol-5calable` in us-east-1
    + adds the description to the snapshot
    + run [fsfreeze](http://linux.die.net/man/8/fsfreeze) to stop writes to `/data` during the snapshot
    + writes the snapshot id to STDOUT for `ec2-replicate-snapshots`
    + writes errors to STDERR which are sent to syslog via logger
* calls `ec2-replicate-snapshots` to do the following
    + reads the snapshot id from STDIN
    + waits for the snapshot to complete (30 minutes)
    + starts the copy to another region, in this case us-west-2 (or retries for 30 minutes if too many copies are in-flight)
    + waits for the copy to complete (6 hours)
* calls `ec2-expire-snapshots` twice to do the following
    + expire snapshots in us-east-1 for `vol-5calable`
        - keeping 2 days of hourlies
        - keeping 2 weeks fo dailies
        - keeping 2 months of weekies
        - keeping 2 years fo monthlies
    + expire snapshots in us-west-2 for volumes that have the tag with key MasterVolumeId and value `vol-5calable`
        - keeping 2 days of hourlies
        - keeping 2 weeks fo dailies
        - keeping 2 months of weekies
        - keeping 2 years fo monthlies

## tl;dr

```
ec2-consistent-snapshot $SNAPARGS | ec2-replicate-snapshots $REPLARGS
ec2-expire-snapshots $EXPIREARGS $PRIMARYREGION
ec2-expire-snapshots $EXPIREARGS $SECONDARYREGION
```

### ToDos

* Fail louder. Need to know when DR isn't right.
* Move timeouts into command line arguments.

### License

Copyright 2013 Corsis
http://www.corsis.com/

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


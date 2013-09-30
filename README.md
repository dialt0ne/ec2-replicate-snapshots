## ec2-replicate-snapshots

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


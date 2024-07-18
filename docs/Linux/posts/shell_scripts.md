## path priority

when you set
```shell
$ export PATH="/path/to/my/dir:$PATH"
```

the priority will be:

- `/path/to/my/dir`

- rest of `$PATH`
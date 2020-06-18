## Ruby utf-8 encoding problem

### tldr

When using ruby in a docker you should set `LANG` env variable to `C.UTF-8` (or other value with `UTF-8` encoding).

### What is the problem?

Ruby interpreter by default wants to use utf8 encoding. To check this run following command on your local machine (not docker):

```
ruby -e 'puts STDIN.external_encoding'
```

Output:

```
UTF-8
```

In typically configured machine it prints `UTF-8`.

Now run the same command but using one of docker ruby images:

```
docker run --rm -it ruby:2.7.0 ruby -e 'puts STDIN.external_encoding'
```

Output:

```
US-ASCII
```

The result is that your program may blow up when utf characters are used:

```
docker run --rm -it ruby:2.7.0 ruby -e 'puts "ąęć".size'
```

Output:

```
-e:1: invalid multibyte char (US-ASCII)
-e:1: invalid multibyte char (US-ASCII)
-e:1: invalid multibyte char (US-ASCII)
-e:1: invalid multibyte char (US-ASCII)
-e:1: invalid multibyte char (US-ASCII)
-e:1: invalid multibyte char (US-ASCII)
```

It happened because ruby interpreter doesn't set utf8 encoding internally when locale is set to non utf encoding.

You can check locale settings with this command:

```
docker run --rm -it ruby:2.7.0 locale
```

Output:

```
LANG=
LANGUAGE=
LC_CTYPE="POSIX"
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=
```

Here are some links that explains the problem in more details:

* https://github.com/docker-library/ruby/issues/45
* https://oncletom.io/2015/docker-encoding/
* https://github.com/docker-library/docs/pull/689/files

### Solution

Set `LANG` env variable to `C.UTF-8` or other value with `UTF-8` encoding.
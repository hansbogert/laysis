# Laysis

Point out Docker *lay*ers using Git history analy*sis*, which are costly during
build in terms of cache invalidation and layer size.

## Basic underlying mechanisms

- Get map of
    - Dockerfile entry
        - its layer
        - its files
- Get frequency of edits of all files of last X commits

## structures

Dockerfile {
[]Layers { size
[]files } }

Git {
[]files { frequency } }

## Calculating cost

- From top layer to lower layer
    - calculate size * ( max(frequency,1 ))
    - prev layer + (size * freq)
    - ...

## Technical difficulties

1. Getting Dockerfile info to be able to get a list of files related to a file.

2. Correlate file from repo to file in a layer. Which we'll discuss in Is
   metadata saved in the dockerfile somehow? Should we parse the ADD and COPY
   files ourselves?

We'll discuss the problems in the subsections

### Getting Dockerfile file info

1. Get history + layers in
   ```
   /var/lib/docker/image/overlay2/imagedb/content/sha256/[imagesha] | jq .history
   ```
2. Get layers from docker inspect:
   ```shell
   docker inspect laysis-example | jq .[].GraphDriver
    ```
   The lower dirs and upper dir form the image as a stack as follows

    1. upper dir
    2. first entry in LowerDir
    3. 2nd entry in LowerDir
    4. .. etc

### Correlating files from Docker and Git

Preliminary investigation shows that we have to 2 cases when reasoned from the
perspective of a Dockerfile, namely,

1. COPY
2. ADD

Explicit copy is relatively easy,

```Dockerfile
COPY foobar /foobar2
```

We can keep a file mapping, in this case from `/foobar2` to `foobar` (note the
reversed order in contrast to the example's `COPY`
directive). During the cost calculating algorithm we should then be able to get
git files and their frequency by their COPY'd name.

More elaborate, but similar logic can be used for,

```Dockerfile
COPY foobar-directory/ /foobar-dst
```

enumerate all files under `foobar-directory`, and map them correspondingly to a
location under `/foobar-dst`

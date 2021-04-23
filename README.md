## Expected behavior

The DESCRIPTION file lists four remote packages:

```
Imports:
    htmltools,
    learnr,
    parsons,
    sortable
Remotes: 
    rstudio/htmltools@11cfbf3b331387922c0144203cf6e947c8bdaee9,
    rstudio/learnr@428d9af1506447f68890724bfc619ed1f2e7cfc0,
    rstudio/parsons@42d0bfb8f23a3c8048cc4c1fbca95a0ef1e0bca3,
    rstudio/sortable@cdde09d915ee0bc73353b079c7afbed8cb70ab4a
```

I expect that running 

```r
renv::install()
renv::snapshot()
```

will result in those four packages and their dependencies being added to the lockfile. Specifically, I expected that each of these four packages would specifically use the remote reference specified in `Remotes`.

## Observed Behavior

The requested version of sortable is not installed, instead the version at `rstudio/sortable@HEAD` is installed.

```
# renv_install.log
Retrieving 'https://api.github.com/repos/rstudio/parsons/tarball/42d0bfb8f23a3c8048cc4c1fbca95a0ef1e0bca3' ...
	OK [downloaded 305.9 Kb in 0.9 secs]
Retrieving 'https://api.github.com/repos/rstudio/sortable/tarball/db2c220b289890d3cd65efcf3a0ab682681e2989' ...
	OK [downloaded 1.6 Mb in 1.1 secs]
	
```

```
# renv_snapshot.log

# GitHub =============================
- htmltools     [* -> rstudio/htmltools@11cfbf3b331387922c0144203cf6e947c8bdaee9]
- learnr        [* -> rstudio/learnr@428d9af1506447f68890724bfc619ed1f2e7cfc0]
- parsons       [* -> rstudio/parsons@42d0bfb8f23a3c8048cc4c1fbca95a0ef1e0bca3]
- sortable      [* -> rstudio/sortable]
```

parsons references `rstudio/sortable` in its `Remotes` field and also imposes a version requirement, but the requested version of sortable does satisfy this requirement.

```
Imports:
    sortable (>= 0.3.1),
    ...
Remotes: 
    rstudio/sortable,
    rstudio/learnr
```

## One workaround

I've been trying to get around this type of problem by purging packages defined in `Remotes` and then re-installing:

```r
> renv::purge("sortable")
The following packages will be purged from the cache:

	sortable 0.4.4.9000 [Hash: c7615bec113ce0148904988d2a9f1233]

Do you want to proceed? [y/N]: y
* Removed 1 package from the cache.
> renv::install()
The following package(s) have broken symlinks into the cache:

	sortable

Consider re-installing these packages.

Retrieving 'https://api.github.com/repos/rstudio/sortable/tarball/db2c220b289890d3cd65efcf3a0ab682681e2989' ...
	OK [downloaded 1.6 Mb in 0.8 secs]
Installing sortable [0.4.4.9000] ...
	OK [built from source]
> renv::install("rstudio/sortable@cdde09d915ee0bc73353b079c7afbed8cb70ab4a")
Retrieving 'https://api.github.com/repos/rstudio/sortable/tarball/cdde09d915ee0bc73353b079c7afbed8cb70ab4a' ...
	OK [downloaded 1.6 Mb in 0.8 secs]
Installing sortable [0.4.4] ...
	OK [built from source]
> renv::snapshot()
The following package(s) will be updated in the lockfile:

# GitHub =============================
- sortable   [ver: 0.4.4.9000 -> 0.4.4; ref: master -> cdde09d915ee0bc73353b079c7afbed8cb70ab4a; sha: db2c220b -> cdde09d9]
```

### ...that doesn't really work

But if I purge and install _all_ of the remotes to try to achieve the same result for all of them, the original problem resurfaces, in some form.

Here I purge parsons and sortable and reinstall. While I end up with the desired versions of sortable and parsons, learnr is moved from the ref mentioned in the DESCRIPTION file to a new ref (master).

```
> renv::purge("parsons")
The following packages will be purged from the cache:

	parsons 0.0.3.9000 [Hash: c44975b7d69c9cbb3d724349aeab6f81]

Do you want to proceed? [y/N]: y
* Removed 1 package from the cache.
> renv::purge("sortable")
The following packages will be purged from the cache:

	sortable 0.4.4 [Hash: 2b6be13d10bfbc2811d396282211651c]

Do you want to proceed? [y/N]: y
* Removed 1 package from the cache.
> renv::install(c(
+     "rstudio/sortable@cdde09d915ee0bc73353b079c7afbed8cb70ab4a",
+     "rstudio/parsons@42d0bfb8f23a3c8048cc4c1fbca95a0ef1e0bca3"
+ ))
The following package(s) have broken symlinks into the cache:

	parsons, sortable

Consider re-installing these packages.

Retrieving 'https://api.github.com/repos/rstudio/sortable/tarball/cdde09d915ee0bc73353b079c7afbed8cb70ab4a' ...
	OK [downloaded 1.6 Mb in 0.7 secs]
Retrieving 'https://api.github.com/repos/rstudio/parsons/tarball/42d0bfb8f23a3c8048cc4c1fbca95a0ef1e0bca3' ...
	OK [downloaded 305.9 Kb in 0.5 secs]
Installing sortable [0.4.4] ...
	OK [built from source]
Installing parsons [0.0.3.9000] ...
	OK [built from source]
> renv::snapshot()
The following package(s) will be updated in the lockfile:

# GitHub =============================
- learnr   [ref: 428d9af1506447f68890724bfc619ed1f2e7cfc0 -> master]
```

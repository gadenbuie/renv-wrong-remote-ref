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

# Example

     # pg_sizes
          'dataplot'  total size:   16MB, of which    table 12.7MB    toast  459KB    indexes  2.5MB    other  8.2KB
             'music'  total size:  5.1GB, of which    table  3.1GB    toast  1.1GB    indexes    1GB    other   33KB
            'quotes'  total size:  6.4MB, of which    table  2.8MB    toast  492KB    indexes  3.1MB    other     0B
             'solar'  total size:  537MB, of which    table  348MB    toast  459KB    indexes  189MB    other     0B
     Total size of databases: 5.7GB


    # pg_sizes -s quotes -c -p
    TABLE 'quotes'.'quotes'
      base table size    616 kB
           TOAST size     40 kB
           index size    712 kB
    Number of rows: 3155
      NAME                                            TYPE       SUM_SIZE                AVG_SIZE     SUM_OCTET  AVG_OCTETSIZE   COMPR
      quote                                           text         432090 (   0 MB)         137.0        417 kB         135.3    101%
      quoter                                          text          32371 (   0 MB)          10.3         29 kB           9.3    110%
      source                                          text          16779 (   0 MB)           5.3         13 kB           4.3    123%
      viewed                                       integer           6252 (   0 MB)           4.0
      proposed                                     boolean           3155 (   0 MB)           1.0
    Total size of databases: 6.4MB

 Where the last few columns are there to point out something like disk use versus apparent size - which is different mainly due to overheads and TOAST compression
 - DISK_SIZE is space taken on disk (via pg_column_size)
 - DISK_AVG is that divided by row count
 - DATA_SIZE is the total uncompressed size (via octet_length, only applies to some types)
 - DATA_AVG is that divided by row count
 - COMPR is DISK_SIZE / DATA_SIZE (indicates overheads, and possible TOAST compression)
 
# Options

```
Usage: pg_sizes [options]

Options:
  -h, --help            show this help message and exit
  -d DBSUBSTR, --dbname-substring=DBSUBSTR
                        Only report databases containing given substring. Can
                        supply multiple with commas.
  -p, --precise-rowcount
                        By default we get the row count estimate via VACUUM
                        statistics, which can make the column sizes a little
                        imprecise. This forces a count(*), which is slower but
                        precise.
  -c, --column-sizes    Estimate size use per column. NOTE: this basically
                        reads all data, so is slow and will mess with your
                        caches
  -t TABSUBSTR, --table-substring=TABSUBSTR
                        Only show column sizes for tables containing given
                        substring. Can supply multiple with commas.
```


# Notes

The database summary comes entirely from data in [pg_class](https://www.postgresql.org/docs/9.6/catalog-pg-class.html) for each database. 

The column stuff uses [pg_column_size and octet_length](https://www.postgresql.org/docs/9.6/functions-admin.html#FUNCTIONS-ADMIN-DBOBJECT)
(and some assumptions about how those relate to TOAST and indices, which is probably at least part of why they seem a little different)


# Caveats

Proof of concept version, contains various hardcoded assumptions.
I still need to reread the docs on the various size functions to ensure it means what I think it means.

The column summaries need to read a lot of table data. Which is slowish, and for databases larger than RAM will probably shred your nicely warmed caches.

Needs access to each database, so:
- assumes we connect as postgresql role `postgres`
- assumes that's trusted on localhost via pg_hba (so no username)
- assumes that's the admin and can read everything

You're running someone else's code on your database.


# TODO / CONSIDER

Output as JSON or something else parsable

A 'forced index warming' mode or such, because we have code that matches filenames to relations, it wouldn't be a stretch to have 

It may be useful to show disk use rather than apparent size, by stat()s to the filesystem. The code for it is commented out because it has to assume you have filesystem permissions, which is a point of failure you don't have using pg_class.

Clean up code, do some things more properly.

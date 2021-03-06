# Woods Hole CMG Observation Data

This is a python script to download and process NetCDF data from `http://geoport.whoi.edu/thredds/catalog/usgs/data2/emontgomery/stellwagen/Data/catalog.html` and turn it into CF-1.6 compliant NetCDF DSG files. There is also a script to convert the CF files into "device" file that Axiom uses to submit data to NCEI and load into the portal system.

### Installation

```bash
$ git clone https://github.com/axiom-data-science/usgs-cmg-portal.git
$ cd usgs-cmg-portal/woods_hole_obs_data
$ conda install -c conda-forge -c axiom-data-science --file requirements.txt
```

### CSV Metadata file

The script uses a CSV metadata file to get information about each project. There is a CSV file called `project_metadata.csv` that is included with the project.  The `project_name` and `catalog_xml` columns are **REQUIRED**.  Any other column names are added as global attributes to each file produced from the specifed `catalog_xml` link.

##### Ignoring Rows

You can prefix a row in the CSV file with the `#` character to ignore the row when processing.

##### Example

Below, the global attributes `contributer_name`, `project_title`, and `project_summary` are added to the resulting NetCDF files and the `BARNEGAT` project is ignored.

```
project_name,contributor_name,project_title,project_summary,catalog_xml
"ARGO_MERCHANT","B. Butman","Argo Merchant Experiment","A moored array deployed after the ARGO MERCHANT ran aground onNantucket Shoals designed to help understand the fate of the spilled oil.","http://geoport.whoi.edu/thredds/catalog/usgs/data2/emontgomery/stellwagen/Data/ARGO_MERCHANT/catalog.xml"
#"BARNEGAT","N. Ganju","Light attenuation and sediment resuspension in Barnegat Bay New Jersey"," Light attenuation is a critical parameter governing the ecological function of shallow estuaries.  Near-bottom and mid-water observations of currents, pressure, chlorophyll, and fDOM were collected at three pairs of sites sequentially at different locations in the estuary to characterize the conditions.","http://geoport.whoi.edu/thredds/catalog/usgs/data2/emontgomery/stellwagen/Data/BARNEGAT/catalog.xml"
"BUZZ_BAY","B. Butman","Currents and Sediment Transport in Buzzards Bay","Investigation of the near-bottom circulation in Buzzards Bay and consequent transport of fine-grained sediments that may be contaminated with PCBs from inner New Bedford Harbor.","http://geoport.whoi.edu/thredds/catalog/usgs/data2/emontgomery/stellwagen/Data/BUZZ_BAY/catalog.xml"
...
```

### Running

```bash
$ python collect.py --help
usage: collect.py [-h] -o [OUTPUT] [-d] [-f FOLDER]
                  [-p [PROJECTS [PROJECTS ...]]] [-l [FILES [FILES ...]]]
                  [-c [CSV_METADATA_FILE]] [-s SINCE]

optional arguments:
  -h, --help            show this help message and exit
  -o [OUTPUT], --output [OUTPUT]
                        Directory to output NetCDF files to
  -d, --download        Should we download a new set of files or use the files
                        that have already been downloaded? Useful for
                        debugging. Downloaded files are never altered in any
                        way so you can rerun the processing over and over
                        without having to redownload any files.
  -f FOLDER, --folder FOLDER
                        Specify the folder location of NetCDF files you wish
                        to translate. If this is used along with '--download',
                        the files will be downloaded into this folder and then
                        processed. If used without the '--download' option,
                        this is the location of the root folder you wish to
                        translate into NetCDF files.
  -p [PROJECTS [PROJECTS ...]], --projects [PROJECTS [PROJECTS ...]]
                        Specific projects to process (optional).
  -l [FILES [FILES ...]], --files [FILES [FILES ...]]
                        Specific files to process (optional).
  -c [CSV_METADATA_FILE], --csv_metadata_file [CSV_METADATA_FILE]
                        CSV file to load metadata about each project from.
                        Defaults to 'project_metadata.csv'.
  -s SINCE, --since SINCE
                        Only process data modified since (utc) - format YYYY-
                        MM-DDTHH:II
```

### Examples

##### Download and create CF-1.6 NetCDF files for a single project
```bash
$ python collect.py --download \
                    --projects SOUTHERN_CAL \
                    --output=./output/
```

##### Download and create NetCDF files for multiple projects
```bash
$ python collect.py --download \
                    --projects SOUTHERN_CAL DIAMONDSHOALS MYRTLEBEACH \
                    --output=./output/
```

##### Download and create NetCDF files for a single file
```bash
$ python collect.py --download \
                    --projects EUROSTRATAFORM \
                    --files "7031adc-a.nc" \
                    --output=./output/
```


##### Specify a different CSV metadata file
```bash
$ python collect.py --download \
                  --projects SOUTHERN_CAL DIAMONDSHOALS MYRTLEBEACH \
                  --output=./output/ \
                  --csv_metadata_file /some/path/to/your/file.csv
```

##### Reprocess already downloaded files with new CSV metadata
```bash
$ python collect.py --projects SOUTHERN_CAL \
                    --folder [path_to_downloaded_files] \
                    --output=./output/
```


##### Reprocess project specific files modified since a date/time
```bash
$ python collect.py --download \
                    --projects SOUTHERN_CAL \
                    --since 2016-02-06T00:00 \
                    --output=./output/
```

##### Reprocess files modified since 24 hours ago (linux only)
```bash
$ python collect.py --download \
                    --since $(date -u -d '-1 day' +%Y-%m-%dT%H:%M) \
                    --output=./output/
```

# Devices

This translates data in CF-1.6 format (that `collect.py` produces) at http://geoport.whoi.edu/thredds/catalog/usgs/data2/emontgomery/stellwagen/CF-1.6/catalog.html into Device NetCDF files.

* Creates a file for each data variable that has a 'coordinates' attribute
* Changes from positive up to positive down
* Renames vertical axis variable name from `z` to `height`
* Combines multiple files of the same station each with different depths into one file (_d1, _d2, _d3, etc.)

### Running

```bash
$ python devices.py --help
usage: devices.py [-h] -o [OUTPUT] -f [FOLDER] [-d]
                  [-p [PROJECTS [PROJECTS ...]]] [-l [FILES [FILES ...]]]
                  [-s SINCE]

optional arguments:
  -h, --help            show this help message and exit
  -o [OUTPUT], --output [OUTPUT]
                        Directory to output NetCDF files to
  -f [FOLDER], --folder [FOLDER]
                        Folder that the CF1.6 NetCDF files from USGS CMG
                        THREDDS server were downloaded to
  -d, --download        Should we download a new set of files or use the files
                        that have already been downloaded? Useful for
                        debugging. Downloaded files are never altered in any
                        way so you can rerun the processing over and over
                        without having to redownload any files.
  -p [PROJECTS [PROJECTS ...]], --projects [PROJECTS [PROJECTS ...]]
                        Specific projects to process (optional).
  -l [FILES [FILES ...]], --files [FILES [FILES ...]]
                        Specific files to process (optional).
  -s SINCE, --since SINCE
                        Only process data modified since (utc) - format YYYY-
                        MM-DDTHH:II
```

### Examples

```bash
$ python devices.py -p CALCSTWET --folder ./cf -o ./devices
```

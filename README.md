# VP_Extraction

### THIS REPO IS IN BETA STAGE, AND REQUIRES FURTHER TESTING BEFORE USE ON LARGE JOBS

Repo for the Vertical Profile generation code, used in the DRUID, BioDar and PESTdar projects

This repo contains a refactored version of the QVP (quasi-vertical profile) extraction code which has been extended to allow static CVP (column vertical profile) extraction. Code has also been added that allows dynamic CVP extraction, however this code is not yet executable. Also included is a run script to farm out the many many simulations required for CVP extraction

The program has also had functionality added to be able to read ODIM formatted HDF5 files. Due to some issues with assumptions made by the ODIM reading function in pyart, a modified version of the read_odim_h5 function has been included in the program which subsets the input file by pulse type and time.This is necessary as the pyart ODIM reader is capable of reading ODIM files generated by the [Nimrod2ODIM](https://github.com/cemac/Radar_ODIM_conv) script, however aggregated files generated by the [hdf5 aggregator](https://github.com/ncasuk/nimrod_hdf5_aggregator) are not read properly due to the aggregated nature of the files themselves. While this is an acceptable fix, it may not be entirely future proof. There may also be an argument for including these changes in a PR to the ARM-pyart repo.

Included in this repo is an environment file which can be used for running the program using python 3.8

## Usage

If running a big extraction job, the script `RunVP.sh` should be used. Usage for this is as follows:

Usage: ./RunVP.sh -s <<YYYYmmdd>> -e <<YYYYmmdd>> -m <<CVP|QVP>> [-h] [-v]

Required Arguments:
  -s : start date for vp extraction
  -e : end date for vp extraction
  -m : VP mode, either CVP or QVP (case sensitive)

Options:
  -h : Show this usage helper
  -v : Run programs in verbose mode

The dictionaries containing the different sites, and the input/output paths used, are given in the vp_params.py file, with different dictionaries for cvp and qvp. These dictionaries contain all the parameters for each site. If editting this file, all fields should remain in the dictionary as the VP_Main.py file expects certain keys dependant on the VP mode.

If running the program on its own, the program can be invoked with:

`python vp_extraction.py profile_type from_directory to_directory -s eventT000000 -e eventT235959 [OPTIONALS]`

Provided arguments should be profile type, input directory (`from_directory`), and the output directory (`to_directory`).

It is assumed that the events subdirectories are in the input directory.

For QVP extraction the name of the output file will be generated automatically based on the date from the start (-s) option and on the elevation angle. It will look like 20170517_QVP_20.0deg.nc and will be placed in the output directory at the end of the run. For Static CVP extraction the filename will have the form <column_position>\_<column_radius>km\_<start_date>.nc such as, for example, `Rothamsted_2.5km_20180214.nc`.

Options
The possible options in the function are split into three categories - general, QVP specific and CVP specific. The general options include:
	   -d, --debug : output debug messages during program execution (currently unused);
	   -v, --verbose: print messages (debug,info,warning,...) about the program execution to the console (stderr);
	   -s , --start time (included):  date and time in format: yyyymmdd'T'HHMMSS;
	   -e , --end time (included): date and time in format: yyyymmdd'T'HHMMSS;
	   -f , --fields for QVP generation of variables for QVP calculation;
	   -z , --zoom time-interval: Time interval to zoom, Formatted as (yyyymmdd'T'HHMMSS,yyyymmdd'T'HHMMSS);
	   -m , --met-office: Flag to indicate that the source of the data is a NIMROD ODIM-formatted HDF5 file;

The QVP options include:
	   -l , --elevation: The elevation at which to calculate the QVP. Required argument for QVP extraction, and no default value. It is assumed that the elevation angle is in the list of elevations in the volume radar data file;
	   -c , --count-threshold: Minimal number of points for the mean value calculation at each range;
	   -b , --azimuth bounds to exclude: Azimuths that should be excluded in the mean value calculation. Format [start1,end1;start2,end2];

The CVP options include:
	   -r , --column-radius: The radius of the column for CVP extraction;
		 -a , --column_latitude: The latitude of the column for CVP extraction;
		 -o , --column-longitude: The longitude of the column for CVP extraction;
		 -p , --column-position: The name of the static position of the column (station or lightning trap) for CVP extraction;

-s and -e options allow to make a VP over several days, providing input directory is the same in all cases and assuming that where not processing a NIMROD file that each day (event - YYYYmmdd) is inside a subdirectory with a corresponding name (e.g. ./20170203/ and ./20170204/ in the directory ./). For the 90 deg. elevation subfolder ver/ is assumed after the event subdirectory (e.g. ./20170203/ver/). The format of these options is: '20170517T000000'. First part before “T” is the date, the second part is the time (HHMMSS). Setting up these options to -s $eventT000000 -e $eventT235959, where $event is the date will collect all the nc-files from the event’s subdirectory.

-f option is the list of variables to be included in QVP file. It’s assumed, that all these variables can be found in the input nc-files. (e.g. current default list is ['PhiDP','RhoHV','SQI','W','dBZ','ZDR','KDP','V']) Field list should be provided as a space-separated list (i.e. 'PhiDP RhoHV SQI W dBZ ZDR KDP V')

-z option allows to zoom into a part of the day event. Might be redundant as it was used before adding -s and -e options.

-c option is the count threshold that is used as a minimum number of non-NAN azimuthal values to be used for the calculation of an azimuthal mean. The value used here should be proportional (e.g. 2/3) to the remaining number of azimuthal directions if -z option is used. Another way to ignore it, is to set it to 1, so that any azimuth with at least one non-NaN value will be used.

-a option provides azimuth bounds to exclude from the average calculation. Possible values are from 0 to 359 (e.g. 45,185). This option can be used to remove certain azimuthal directions from the calculation if there is an obstacle or a known artefact influencing the observations. Multiple azimuth exclusions are possible by separating the bounds with a semicolon. For example, if there is an obstruction between 23 and 34 degrees, another between 221 and 255 degrees and a third at only 351 degrees, the bounds can be passed to the program as `-a 23,34;221,255;351,351`

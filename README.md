# Azimuth / Pitch Calculator

## In one sentence

Command line Python script that 1) takes two sequential geotagged 360 images, 2) reads latitude, longitude and altitude, 3) calculates azimuth (or heading) and pitch for source photo, 4) creates a new photo file with azimuth and pitch values embedded in metadata.

## Why we built this

Our [Trek Packs](https://www.trekview.org/trek-pack/) use GoPro Fusion and MAX Cameras in timelapse modes.

The problem, the camera does not write azimuth (or heading, left/right, 360/0) or pitch (up/down, 90, -90) information at capture time.

![Azimuth and pitch schematic](/readme-images/azimuth-altitude-schematic.png)

For 360 photos this is important when rendering them in a 360 viewer (e.g. Google Street View) when setting the initial view. 

Without a value set many default to north (0 degrees) and straight ahead (0 degrees).

For geotagged photos taken in a timelapse, it is possible to provide a fairly accurate estimate of the azimuth and pitch (see: limitations).

![Calculating Pitch from series of geotagged photos](/readme-images/pitch-calculation.png)

![Calculating Azimuth from series of geotagged photos](/readme-images/photo-heading-calculation.png)

For pitch this is done by calculating the verticle angle between the altitude value of source and destination photo.

For azimuth this is done by calculating the horizontal angle between the source photo (latitude/longiture) and the destination photo (latitude/longiture).

The azimuth and pitch value can then be reported in the source photo image metadata fields:

* [EXIF] `GPSImgDirection` (azimuth)
* [EXIF] `GPSPitch` (pitch)
* [EXIF] `CameraElevationAngle` (pitch)
* [XMP] `PoseHeadingDegrees` (azimuth)
* [XMP] `PosePitchDegrees` (pitch)

You can embed this information into the image programmatically using exiftool. For example:

```
exiftool -xmp:PosePitchDegrees="10.2" my_360_image.jpg
```

Azimuth Calculator is a simple Python script that calculates the azimuth of a source image from a destination image.

## How it works

1. User enters a directory of 2 or more panoramic images and defines the order they should be processed
2. The script orders the images in the defined sequence and performs calculations for the source and destination photo
3. The script outputs a new photo file with new azimuth and pitch metadata appended

## Requirements

### OS Requirements

Works on Windows, Linux and MacOS.

### Software Requirements

* Python version 3.6+
* [exiftool](https://exiftool.org/)

### Image Requirements

All images must be geotagged with the following values:

* `GPSLatitude`
* `GPSLongitude`
* `GPSAltitude`
* `GPSDateTime`

If a photo does not contain this infortmation, you will be presented with a warning, and asked wether you wish continue (discard the photos missing this inforamtion from the process).

This software will work with most image formats. Whilst it is designed for 360 photos, it will work with a sequence of traditional flat (Cartesian) images too.

## Quick start guide

### Installation

The following Python packages need to be installed:
* [PyExifTool](https://pypi.org/project/PyExifTool/)
	`python -m pip install PyExifTool`
* [Pandas](https://pandas.pydata.org/docs/)
	`python -m pip install pandas`


[exiftool](https://exiftool.org/) needs to be installed on the system.
If used on Windows, download the stand-alone .exe executable. Rename the .exe file to `exiftool.exe`. Put the .exe file in the same folder as the `azipi.py` file

The `.ExifTool_Config` ([.ExifTool_Config explanation](https://exiftool.org/faq.html#Q11)) needs be in the HOME directory (Mac, Linux) or in the same folder as the `azipi.py`file (Windows)

### Arguements

* c: connection type
	- time (capture time of image)
	- filename
* o: connection order
	- ascending (e.g. 00:01 - 00:10 or A.jpg > Z.jpg)
	- descending (e.g. 00:10 - 00:01 or Z.jpg > A.jpg)
* d: discard: discard images that lack GPS or time tags and continue
* e: exiftool-exec-path
	- path to ExifTool executable (recommended on Windows if [exiftool.exe](https://exiftool.org/) is not in working directory)
* input_directory: directory that contains a series of images
* output_directory: directory to store the newly tagged images

Note: we use [EXIF] `GPSDateTime` not [EXIF] `CaptureTime` values for datetime because many 360 stitching tools rewrite `CaptureTime` as datetime of stiching process not the datetime the image was actually captured. This can cause issues when sorting by time (e.g. images might not be stiched in capture order). Therefore, `GPSDateTime` is more likely to represent the true time of capture.

### Format

`python azipu.py -c [CONNECTION TYPE] -o [CONNECTION ORDER] -d - e [PATH TO EXIFTOOLS] [INPUT PHOTO DIRECTORY] [OUTPUT PHOTO DIRECTORY]`

### Examples

**Calculate azimuth and pitch by ordring photos in ascended time order (recommended)**

Mac, Linux:

`python azimuth-calculator.py -c time -o ascending -d my_360_photos/ my_updated_360_photos/`

Windows example:

"C:\Program Files (x86)\Python37\python.exe" "C:\PATH TO Python file\azipi.py" -c time -o ascending -d -e "C:\PATH TO exiftool\exiftool.exe" "C:\PATH TO INPUT FOLDER\" "C:\PATH TO OUTPUT FOLDER\"

This command is what 99% of people will need. It process the earliest capture to the latest capture (by time).

However, you can also order by filename (`-c filename`) and/or descending order (`-o descending`).

### Output

You will get a new photo file with appended meta data.

The new files will follow the naming convention: `[ORIGINAL FILENAME] _ calculated . [ORIGINAL FILE EXTENSION]`

## Limitations

**Estimation**

Photos in our tours are generally less than 3m apart and our Trek Pack cameras are always facing forward / backwards in the same direction (in a fixed position), we make the assumption that the camera is facing in the direction of the next photo (by time).

Note, this will not always be correct, for example, if camera turns 90° between start and destination photo (e.g turning a corner). In such cases, using this software could result in photos facing the wrong direction (e.g. a brick wall in the 90 example). However, for our outdoor work this is rareley a problem and is considered acceptable.

If you are shooting at a low frame rate and sharply changing direction by a large degree, azimuth calcularot might now be perfect for you use case.

**Final photo**

The last photo in the sequence has no subsequent photo. Therefore, as a simple solution the last photo simply inherits the values of the previous photo in the sequence.

Again, this will not be suitable when precise accuracy is needed.

## Support 

We offer community support for all our software on our Campfire forum. [Ask a question or make a suggestion here](https://campfire.trekview.org/c/support/8).

## License

Azimuth / Pitch Calculator is licensed under a [GNU AGPLv3 License](https://github.com/trek-view/azimuth-pitch-calculator/blob/master/LICENSE.txt).
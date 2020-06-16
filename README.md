# Azimuth / Pitch Calculator

## In one sentence

Command line Python script that 1) takes two sequential geotagged 360 images (source and destination), 2) reads time or filename, and latitude, longitude and altitude, 3) calculates azimuth (or heading) and pitch for source photo, 4) creates a new photo file with azimuth and pitch values embedded in metadata.

## Why we built this

Our [Trek Packs](https://www.trekview.org/trek-pack/) use GoPro Fusion and MAX Cameras in timelapse modes.

The problem; the cameras do not write azimuth (or heading, left/right, 360/0) or pitch (up/down, 90, -90) information at capture time in timelapse mode.

![Azimuth and pitch schematic](/readme-images/azimuth-altitude-schematic.png)

For 360 photos this is can be important when rendering them in a 360 viewer (e.g. Google Street View) when setting the initial view. Without a value, many default to value for north (0 degrees) and straight ahead (0 degrees).

Azimuth / Pitch Calculator is a simple Python script that calculates the azimuth and pitch of a source image using the next (destination) image in a timelapse.

## How it works

### Overview

1. User enters a directory of 2 or more panoramic images and defines the order they should be processed (either by time or filename, in ascending or descending order)
2. The script orders the images in the defined sequence and performs calculations to work out azimuth nd pitch using the source and destination photo
3. The script outputs a new photo file with new azimuth and pitch metadata appended and moves onto the next image in the designated order to perform step 2
4. The script continues this loop until all images have been processed. For the final image (where there is not destination photo), the azimuth and pitch is inherited from the last image in the sequence.

### The Details 

For geotagged photos taken in a timelapse, it is possible to provide a fairly accurate estimate of the azimuth and pitch (see: limitations) because timelapses are typically shot in ascending time order (00:00:00 > 00:00:05 > 00:00:10) at set intervals (e.g. one photo every 5 seconds). 

![Calculating Pitch from series of geotagged photos](/readme-images/pitch-calculation.png)

For pitch this is done by calculating the vertical angle between the `GPSAltitude` value of source and destination photo.

![Calculating Azimuth from series of geotagged photos](/readme-images/photo-heading-calculation.png)

For azimuth this is done by calculating the horizontal angle between the source photo (`GPSLatitude`/`GPSLongitude`) and the destination photo (`GPSLatitude`/`GPSLongitude`).

using `distance_mtrs` and `time_sec` it is possible to calculate speed between two photos (speed = `distance_mtrs` / `time_sec`

The azimuth and pitch value are then reported in the source photo image metadata fields using the following standard tags:

* [EXIF] `GPSImgDirection` (azimuth) (measure: degrees)
* [EXIF] `GPSPitch` (pitch) (measure: degrees)
* [EXIF] `GPSSpeed` (speed) (measure: km/h)
* [EXIF] `GPSSpeedRef` (speed measurement, always `k` for km/h)
* [EXIF] `CameraElevationAngle` (pitch) (measure: degrees)
* [XMP] `PoseHeadingDegrees` (azimuth) (measure: degrees)
* [XMP] `PosePitchDegrees` (pitch) (measure: degrees)

_[Read more about EXIF and XMP standards here](https://github.com/trek-view/360-camera-metadata)._

This information is then embedded programmatically using [exiftool](https://exiftool.org/). For example:

```
exiftool -xmp:PosePitchDegrees="10.2" my_360_image.jpg
```

### Limitations / Considerations

**Estimations**

Photos in our (Trek View) tours are generally less than 3m apart and our Trek Pack cameras are always facing forward / backwards in the same direction (in a fixed position). Azimuth / Pitch Calculator therefore makes the assumption that the camera is facing in the direction of the next photo (as defined in CLI arguments).

Note, this will not always be correct, for example, if camera turns 90 degrees between start and destination photo (e.g turning a corner). In such cases, using this software could result in photos facing the wrong direction and causing visual issues (e.g. facing a brick wall if turning 90 degrees around a city block). However, for our outdoor work this is rarely a problem and is considered acceptable.

If you're shooting at a low frame rate, sharply changing direction, or holding your camera at different angles (e.g holding in your hand), Azimuth / Pitch Calculator will not be a good fit for you.

**Discarded images**

This script allows you to discard images that don't match a certain criteria (for example, you want to space them a minimum of 10 meters apart) and will lead to a larger level of inaccuracy for estimations.

In cases where more images are discarded the source and destination photo used for calculations might therefore be very far apart, and thus less likely for the source photo to be facing the destination photo.

**Final photo**

The last photo in the timelapse (sequence) has no subsequent photo. Therefore, as a simple solution the last photo simply inherits the values of the previous photo in the sequence.

This 'best-guess' will not be suitable when precise accuracy is needed.

**Missing GPS**

Azimuth / Pitch Calculator assumes all images are correctly geotagged with GPS information.

If an image is not geotagged for some reason, you can still use Azimuth / Pitch Calculator but it will lead to a larger level of inaccuracy for estimations. This is because images with no GPS information are discarded. In cases where there has been loss of GPS information for a long period of time, the source and destination photo used for calculations might therefore be very far apart, and thus less likely for the source photo to be facing the destination photo.

## Requirements

### OS Requirements

Works on Windows, Linux and MacOS.

### Software Requirements / Installation

The following software / Python packages need to be installed:

* Python version 3.6+
* [Pandas](https://pandas.pydata.org/docs/): `python -m pip install pandas`
* [PyExifTool](https://pypi.org/project/PyExifTool/): is used as a package as well. This package is provided within this repo with the `exiftool.py` content being the content of a specific commit to address Windows related issues.
* [exiftool](https://exiftool.org/) needs to be installed on the system. If used on Windows, download the stand-alone .exe executable. Rename the .exe file to `exiftool.exe`. Put the .exe file in the same folder as the `azipi.py` file

The `.ExifTool_Config` ([.ExifTool_Config explanation](https://exiftool.org/faq.html#Q11)) needs be in the HOME directory (Mac, Linux) or in the same folder as the `azipi.py`file (Windows)

### Image Requirements

All images must be geotagged with the following values:

* `GPSLatitude`
* `GPSLongitude`
* `GPSAltitude`
* `GPSDateTime` OR (`GPSDateStamp` AND `GPSTimeStamp`) OR `originalDateTime`

If a photo does not contain this information, you will be presented with a warning, and will be forced to explicitly discard (`-d`) images.

This software will work with most image formats. Whilst it is designed for 360 photos, it will work with a sequence of traditional flat (Cartesian) images too.

## Quick start guide

### Command Line Arguments

* c: connection type (optional: default is time)
	- timegps (`GPSDateTime` of image)
	- timecapture (`originalDateTime` of image)
	- filename (optional: default is ascending)

_A note on connection types. Generally you should join by time unless you have a specific use-case. Filename will join the photo to the next photo in ascending alphabetical order. We recommend using `timegps` ([EXIF] `GPSDateTime`) not `timecapture` ([EXIF] `originalDateTime`) unless you are absolutely sure `originalDateTime` is correct. Many 360 stitching tools rewrite `originalDateTime` as datetime of stitching process not the datetime the image was actually captured. This can cause issues when sorting by time (e.g. images might not be stitched in capture order). Therefore, `GPSDateTime` is more likely to represent the true time of capture._

* o: connection order
	- ascending (e.g. 00:01 - 00:10 or A.jpg > Z.jpg)
	- descending (e.g. 00:10 - 00:01 or Z.jpg > A.jpg)
* d: discard: discard images that lack GPS or time tags and continue (required: if no GPS data in image)
* e: exiftool-exec-path (optional)
	- path to ExifTool executable (recommended on Windows if [exiftool.exe](https://exiftool.org/) is not in working directory)
* input_directory: directory that contains a series of images
* output_directory: directory to store the newly tagged images

### Format

```
python azipi.py -c [CONNECTION TYPE] -o [CONNECTION ORDER] -d - e [PATH TO EXIFTOOLS] [INPUT PHOTO DIRECTORY] [OUTPUT PHOTO DIRECTORY]
```

### Examples

**Calculate azimuth and pitch by ordering photos in ascended time order and discard images without valid GPS (recommended)**

This command is what 99% of people will need. It process the earliest capture to the latest capture (by time).

However, you can also order by filename (`-c filename`) and/or descending order (`-o descending`), if needed.

_Mac/Linux_

```
python azimuth-calculator.py -c timegps -o ascending -d INPUT_DIRECTORY OUTPUT_DIRECTORY
```

_Windows_

```
python azimuth-calculator.py -c timegps -o ascending -d "INPUT_DIRECTORY" "OUTPUT_DIRECTORY"
```

### Output

If successful an output similar to that shown below will be shown:

```
8 file(s) have been found in input directory
Fetching metadata from all images....

Checking metadata tags of all images...
1 images dropped. "DISCARD" is True.

Calculating differences of time, distance, azimuth and pitch between images...
Writing metadata to EXIF & XMP tags of qualified images...

Cleaning up old and new files...
Output files saved to C:\Users\david\azimuth-pitch-calculator-master\OUTPUT

Metadata successfully added to images.
```

You will get a new photo file with appended metadata.

The new files will follow the naming convention: `[ORIGINAL FILENAME] _ calculated . [ORIGINAL FILE EXTENSION]`

For example, `INPUT/MULTISHOT_9698_000000.jpg` >> `OUTPUT/MULTISHOT_9698_000000_calculated.jpg`

## FAQ

**I get the error Missing metadata key: GPSDateTime. How to continue?**

You will receive the following error if one of your images contains no GPS metadata and the `-d` argument is not included when running the script:

```
Missing metadata key: GPSDateTime


Consider using the "-d" option to discard images missing required metadata keys
```

To fix, simply run the script with the `-d` argument included.

**How can I check the metadata in the image?**

You can use exiftool (which will already be installed) to check the metadata.

This skeleton command will output all the metadata in the specified image:

```
exiftool -G -s -b -j -a -T [PATH OF IMAGE TO CHECK] > OUTPUT.json
```

It will give a complete JSON document. Here's a snippet of the output:

```
  "XMP:PoseHeadingDegrees": {
    "id": "PoseHeadingDegrees",
    "table": "XMP::GPano",
    "val": 177.3467244748772
  },
```

## Support 

We offer community support for all our software on our Campfire forum. [Ask a question or make a suggestion here](https://campfire.trekview.org/c/support/8).

## License

Azimuth / Pitch Calculator is licensed under a [GNU AGPLv3 License](https://github.com/trek-view/azimuth-pitch-calculator/blob/master/LICENSE.txt).
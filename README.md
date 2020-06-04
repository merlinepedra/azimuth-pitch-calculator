# Azimuth / Pitch Calculator

## In one sentence

Command line Python script that 1) takes two sequential geotagged 360 images, 2) reads latitude, longitude and altitude, 3) calculates azimuth (or heading) and pitch for source photo, 4) creates a new photo file with azimuth and pitch values embedded in metadata.

## Why we built this

Our [Trek Packs](https://www.trekview.org/trek-pack/) use GoPro Fusion and MAX Cameras in timelapse modes.

The problem, the camera does not write azimuth (or heading) or pitch (vertical direction) information at capture time.

![Azimuth schematic](/readme-images/azimuth-altitude-schematic.png)

For 360 photos this is important when rendering them in a 360 viewer when setting the initial view. 

Without a value set many default to north (0 degrees).

For geotagged photos taken in a timelapse, it is possible to provide a fairly accurate estimate of the azimuth (see: limitations).

![Calculating Azimuth from series of photos](/readme-images/photo-heading-calculation.png)

This is done by calculating the angle between the source photo (latitude/longiture) and the destination photo (latitude/longiture).

The azimuth and pitch value can be reported in standard image metadata fields:

* [EXIF] GPSImgDirection
* [EXIF] GPSPitch
* [EXIF] GPSImgDirectionRef
* [EXIF] CameraElevationAngle
* [XMP] PoseHeadingDegrees
* [XMP] PosePitchDegrees

Doing this manually 

Azimuth Calculator is a simple Python script that calculates the azimuth of a source image from a destination image.

## How it works

1. 

final photo inherits previous heading

## Requirements

TODO

### Software Requirements

TODO

### Image Requirements

This software will work with most image formats. Whilst it is designed for 360 photos, it will work with a sequence of traditional flat (Cartesian) images too.

_Note; if you want to calculate this information for a video file,_

## Quick start guide

### Arguements

* c: connection type
	- time (capture time of image)
	- filename
* o: connection order
	- ascending (e.g. 00:01 - 00:10 or Z.jpg > A.jpg)
	- descending (e.g. 00:10 - 00:01 or A.jpg > Z.jpg)

### FORMAT

`python azimuth-calculator.py -c [CONNECTION TYPE] -o [CONNECTION ORDER] [INPUT PHOTO DIRECTORY] [OUTPUT PHOTO DIRECTORY]`

### Examples

**Calculate azimuth by ordring photos in descending time order**

`python azimuth-calculator.py -c time -o descending my_360_photos/ my_updated_360_photos/`


## Limitations

Given our photos are generally less than 3m apart and our Trek Pack cameras are always facing forward / backwards in the same direction (in a fixed position), we make the assumption that the camera is facing in the direction of the next photo (by time).

Note, this will not always be correct, for example, if camera turns 90° between start and destination photo (e.g turning a corner). In such cases, using this software could result in photos facing the wrong direction (e.g. a brick wall in the 90 example). However, for our outdoor work this is rareley a problem and is considered acceptable.

If you are shooting at a low frame rate and sharply changing direction by a large degree, azimuth calcularot might now be perfect for you use case.

## Support 

We offer community support for all our software on our Campfire forum. [Ask a question or make a suggestion here](https://campfire.trekview.org/c/support/8).

## License

Geovideo to Geoframes is licensed under an [GNU AGPLv3 License](https://github.com/trek-view/geovideo-to-geoframes/blob/master/LICENSE.txt).
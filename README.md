# gpxFileTool

## Intro
gpxFileTool is a simple tool for manipulating GPX files. While it is generic enough to work with any type of activity, it was developed mainly to process GPX files from *cycling* activities recorded by 		devices such as a Garmin Edge or Wahoo Elemnt bike computer, or by mobile apps such as Strava or RideWithGps.
 
The GPS elevation data in these GPX files can be subject to significant errors, which result in incorrect values for the total elevation gain/loss of the ride, and in incorrect values for the grade level during a climb/descent segment of the ride.  

Having an incorrect value for the total elevation gain/loss simply skews one's own personal statistics.  But the incorrect grade level is a problem when such GPX file is used to control a cycling "smart trainer".  The bogus elevation values can result in spikes in the grade level that make the feeling of the *virtual ride* unrealistic, and in extreme cases it can suddenly **lock up** the smart trainer. Imagine you are pedaling your bike on the trainer at a steady pace while climbing a segment with a moderate 4% grade, when all of a sudden you get a spike that sends the grade level to 14% ... Yikes!

One of the design goals for the gpxFileTool is to allow the user to correct these errors, so that the virtual ride on the smart trainer is more realistic. 

The tool has the following features:

1. Can trim out a range of points.
2. Can smooth out the elevation values.
3. Can limit the min/max grade level.
4. Can filter out optional metrics.
5. Can generate a new GPX file or a CSV file.

Trimming out a range of points is useful to remove such things as "red light", "photo shot", or "nature break" stops during a ride.

Smoothing out the elevation values is the main task when preparing a GPX file for a virtual route. The tool uses a Simple Moving Average (SMA) algorithm, over a configurable range of points, to do the elevation smoothing.

Limiting the min/max grade levels is useful when the user knows *a priori* what those limits are for the given route.

Filtering out optional metrics is useful to remove unwanted sensor data, such as heart rate or cadence.

Being able to generate a CSV file allows the file to be processed by an app such as Excel or LibreOffice, to do data analysis and visualization.

In addition, the tool can read the GPX input file from standard input, and it can write the GPX output file to standard output, so that it can be used in a *pipe* to do multiple operations in one shot.

## About GPX Files

GPX files are plain text files that use XML encoding based on the following [data schema](http://www.topografix.com/GPX/1/1/gpx.xsd). 

In a nutshell, a GPX file contains a "track", which contains one or more "track segments", which contain the actual "track points". Each track point includes the GPS coordinates (latitude, longitude, elevation) plus an optional timing data.  Whether or not the GPX file includes this timing data, is the main difference between a GPX *route* and a GPX *ride*.

Below you can see a clip from a GPX ride showing the general structure of the data:

```xml
  <trk>
    <trkseg>
      <trkpt lat="43.7689000000" lon="-114.2755600000">
        <ele>1960.0000000000</ele>
        <time>2021-03-28T14:17:42.010Z</time>
      </trkpt>
      <trkpt lat="43.7692400000" lon="-114.2753200000">
        <ele>1960.2000000000</ele>
        <time>2021-03-28T14:17:54.744Z</time>
      </trkpt>
          .
          .
          .
      <trkpt lat="43.7889600000" lon="-114.2603500000">
        <ele>2121.9000000000</ele>
        <time>2021-03-28T14:30:30.336Z</time>
      </trkpt>
    </trkseg>
  </trk>
```
The latitude and longitude values are expressed in decimal degrees, the elevation in meters, and the time in UTC.

## Examples

The following examples show how to use the tool.  Running the tool with the option --help will show a "manual page" describing all the options: 

```
$ gpxFileTool --help
SYNTAX:
    gpxFileTool [OPTIONS]
OPTIONS:
    --help
        Show this help and exit.
    --input-file <name>
        Read the input data from the specified file. If not specified
        the input data is read from standard input.
    --max-grade <value>
        Limit the maximum grade to the specified value. The elevation
        values are adjusted accordingly.
    --min-grade <value>
        Limit the minimum grade to the specified value. The elevation
        values are adjusted accordingly.
    --name <name>
        String to use for the <name> tag of the track in the output
        GPX file.
    --output-file <name>
        Write the output data into the specified file. If not specified
        the output data is written to standard output.
    --output-filter <mask>
        A bit mask that specifies the set of optional metrics to be
        suppressed from the output:
            0x01 - Ambient Temperature
            0x02 - Cadence
            0x04 - Heart Rate
            0x08 - Power
    --output-format {csv|gpx}
        Specifies the format of the output data.
    --quiet
        Suppress all warning messages.
    --range <a,b>
        Limit the points to be processed to the range between point
        'a' and point 'b', inclusive.
    --rel-time
        Use relative timestamps in the CSV output.
    --remove-stops <speed>
        Remove any points with a speed below the specified minimum
        speed (in km/s), assuming we were actually stopped at the time.
    --set-speed <speed>
        Use the specified speed value (in km/h) to generate missing
        timestamps in the input GPX file.
    --sma-window <value>
        Size of the window used to compute the Simple Moving Average
        of the elevation values, in order to smooth them out. It must be
        an odd value.
    --start-time <time>
        Start time for the activity (in UTC time). The timestamp of each
        point is adjusted accordingly. Format is: 2018-01-22T10:01:10Z.
    --summary
        Print only a summary of the activity metrics in human-readable
        form and exit.
    --trim
        Trim all the points in the specified range. The timestamps of
        the points after point 'b' are adjusted accordingly, to avoid
        a discontinuity in the time sequence.
    --version
        Show version information and exit.
```

#### Example 1

In this example we read in a GPX file created by a Garmin Edge 520 Plus bike computer, and simply print a summary of its data. The option --quiet is used to suppress all warnings.  The summary includes the total number of track points processed, the duration of the ride, its distance and elevation gain/loss, max/min grade, etc.

```
$ gpxFileTool --input-file SampleGpxFiles/Knights_Ferry_GarminEdge520.gpx --quiet --summary
    numTrkPts: 3635
 numDupTrkPts: 0
numTrimTrkPts: 0
   numElevAdj: 119
         time: 04:41:45
   movingTime: 04:41:39
  stoppedTime: 00:00:06
     distance: 108.6702326136 km
     elevGain: 601.9189828138 m
     elevLoss: 587.8633776238 m
    maxDeltaP: 69.766 m at TrkPt #542 (line #5427) : time = 2615 s, dist = 21.94 km
    maxDeltaT: 58.000 sec at TrkPt #1198 (line #11987) : time = 5346 s, dist = 40.29 km
     maxSpeed: 53.1624730132 km/h at TrkPt #822 (line #8227) : time = 3781 s, dist = 30.37 km
     maxGrade: 550.79% at TrkPt #1772 (line #17727) : time = 7782 s, dist = 49.67 km
     minGrade: -437.40% at TrkPt #1179 (line #11797) : time = 5156 s, dist = 40.26 km
```

This ride included a few stops during which the bike was not moving but the Garmin was still recording GPS data. The subtle errors in the GPS data while the bike was stopped cause some track points to have an inconsistent value of distance and elevation.  Running the tool without the --quiet option will show a bunch of these errors:

```
WARNING: TrkPt at line #2717 has inconsistent distance (0.1909778990) and rise (0.6000022888) values! (speed=0.138 km/h)
```
The very low value of the speed is an indication that the bike was actually stopped at that point.  Running the tool with the option --remove-stops *<speed>* will treat as being **stopped** any point where the speed is below *<speed>*.

```
$ gpxFileTool --input-file SampleGpxFiles/Knights_Ferry_GarminEdge520.gpx --remove-stops 0.5 --quiet --summary
    numTrkPts: 3635
 numDupTrkPts: 0
numTrimTrkPts: 0
   numElevAdj: 2
         time: 04:19:28
   movingTime: 03:57:05
  stoppedTime: 00:22:23
     distance: 108.6454860948 km
     elevGain: 583.7220803293 m
     elevLoss: 587.1999111176 m
    maxDeltaP: 69.766 m at TrkPt #542 (line #5427) : time = 2615 s, dist = 21.94 km
    maxDeltaT: 13.000 sec at TrkPt #1641 (line #16417) : time = 7199 s, dist = 48.14 km
     maxSpeed: 53.1624730132 km/h at TrkPt #822 (line #8227) : time = 3781 s, dist = 30.37 km
     maxGrade: 218.30% at TrkPt #1690 (line #16907) : time = 7427 s, dist = 48.49 km
     minGrade: -437.40% at TrkPt #1179 (line #11797) : time = 5156 s, dist = 40.25 km
```
Notice how the new stoppedTime value increased from 00:00:06 to 00:22:23.

And notice also the absurd values for the maximum and minimum grades. These bogus values are the result of bad GPS elevation data readings. Running the tool with the option --sma-window 7 will smooth out the grade values using a Simple Moving Average (SMA) algorithm over a range of 7 points; i.e. the point in question plus 3 points before and 3 points after:

```
$ gpxFileTool --input-file SampleGpxFiles/Knights_Ferry_GarminEdge520.gpx --remove-stops 0.5 --sma-window 7 --quiet --summary
    numTrkPts: 3635
 numDupTrkPts: 0
numTrimTrkPts: 0
   numElevAdj: 3474
         time: 04:19:28
   movingTime: 03:57:05
  stoppedTime: 00:22:23
     distance: 108.6454860948 km
     elevGain: 617.6527809595 m
     elevLoss: 468.2622134643 m
    maxDeltaP: 69.766 m at TrkPt #542 (line #5427) : time = 2615 s, dist = 21.94 km
    maxDeltaT: 13.000 sec at TrkPt #1641 (line #16417) : time = 7199 s, dist = 48.14 km
     maxSpeed: 53.1624730132 km/h at TrkPt #822 (line #8227) : time = 3781 s, dist = 30.37 km
     maxGrade: 49.98% at TrkPt #1690 (line #16907) : time = 7427 s, dist = 48.49 km
     minGrade: -116.47% at TrkPt #1179 (line #11797) : time = 5156 s, dist = 40.25 km
```

While the smoothing brought down the max grade from 218.30% to 49.98%, the value is still bogus to the point that no human being would be able to climb a road with such a grade.  Knowing *a priori* the maximum grade of the route, we can use the option --max-grade *<value>* to limit (cap) the maximum grade to this specified value:

```
$ gpxFileTool --input-file SampleGpxFiles/Knights_Ferry_GarminEdge520.gpx --remove-stops 0.5 --sma-window 7 --max-grade 8.0 --quiet --summary
    numTrkPts: 3635
 numDupTrkPts: 0
numTrimTrkPts: 0
   numElevAdj: 3474
         time: 04:19:28
   movingTime: 03:57:05
  stoppedTime: 00:22:23
     distance: 108.6454860948 km
     elevGain: 598.4443934391 m
     elevLoss: 467.3053959259 m
    maxDeltaP: 69.766 m at TrkPt #542 (line #5427) : time = 2615 s, dist = 21.94 km
    maxDeltaT: 13.000 sec at TrkPt #1641 (line #16417) : time = 7199 s, dist = 48.14 km
     maxSpeed: 53.1624730132 km/h at TrkPt #822 (line #8227) : time = 3781 s, dist = 30.37 km
     maxGrade: 8.00% at TrkPt #1160 (line #11607) : time = 5111 s, dist = 40.18 km
     minGrade: -116.49% at TrkPt #1179 (line #11797) : time = 5156 s, dist = 40.25 km
```

#### Example 2

There are situations in which one wants to turn a GPX route into a GPX ride.  For example, imagine you rode your bike for a couple of hours and at the end of the ride you realize you forgot to start your bike computer. Doh!  In this case you can use a GPX route editor (such as RideWithGPS) to draw the route you rode, and then add timing data to the GPX route to turn it into a ride, so that it can be uploaded to your Strava account to get distance and elevation gain credits for it.  In this example we take a manually created route, and we turn it into a ride using the current date and time as the activity's start time, and an average speed of 12.0 km/h:

```
$ gpxFileTool --input-file SampleGpxFiles/TrailCreekEoP_RWGPS_Route.gpx --summary
TrkPt at line #13 is missing its time data!
Failed to parse input file
$
$ gpxFileTool --input-file SampleGpxFiles/TrailCreekEoP_RWGPS_Route.gpx --start-time now --set-speed 12.0 > TrailCreekEoP_RWGPS_Ride.gpx
$ gpxFileTool --input-file TrailCreekEoP_RWGPS_Ride.gpx --summary
    numTrkPts: 92
 numDupTrkPts: 0
numTrimTrkPts: 0
   numElevAdj: 0
         time: 00:12:48
   movingTime: 00:12:48
  stoppedTime: 00:00:00
     distance: 2.5610856060 km
     elevGain: 164.3000000000 m
     elevLoss: 2.4000000000 m
    maxDeltaP: 129.624 m at TrkPt #17 (line #76) : time = 246 s, dist = 0.82 km
    maxDeltaT: 38.887 sec at TrkPt #17 (line #76) : time = 246 s, dist = 0.82 km
     maxSpeed: 12.0166019232 km/h at TrkPt #27 (line #116) : time = 301 s, dist = 1.00 km
     maxGrade: 30.87% at TrkPt #60 (line #248) : time = 503 s, dist = 1.68 km
     minGrade: -4.23% at TrkPt #38 (line #160) : time = 367 s, dist = 1.22 km
```

#### Example 3

In this example we instruct the tool to generate a Comma-Separated-Value (CSV) output file, so that the file can be loaded into a spreadsheet app (such as Excel or Libre Office Calc) for further analysis and data visualization:

```
$ gpxFileTool --input-file SampleGpxFiles/Galena_Pass_Northbound.gpx --output-format csv > Galena_Pass_Northbound.csv
```

This [screenshot](https://github.com/elfrances/gpxFileTool/blob/main/Images/Image-1.png) shows the graph of elevation vs. distance for this ride, created from the CSV file.















  






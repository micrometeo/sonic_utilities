# ssbf-lib: An open-source file format and Fortran library for storing and using raw ultrasonic anemometer data

by Patrizia Favaron.

## Core motivation

Three-dimensional anemometers can easily provide fast-sampling (10Hz+), accurate and high-resolution (1cm/s, 0.01°C) wind and temperature data whose processing (for example using _eddy covariance_ and Monin-Obukhov similarity theory) can provide both high-resolution mean wind and mean temperature information, along with turbulence indicators like friction velocity, turbulent sensible heat flux or Obukhov length.

These data may in turn be used as a basis for quantifying or predicting the "dispersivity" of lower atmosphere (the "Planetary Boundary Layer"), that is, the tendency of air in contact or close to the Earth surface to dilute and transport trace gases (e.g. human-generated pollutants) and particulates (PM-x, pollens, spores, tiny animals).

However, ultrasonic anemometer data files tend to be much larger than their electro-mechanical anemometer counterparts: the necessity then arises to deal with massive data sets efficiently (a measurement campaign may produce many gigabytes of raw data).

## Relevant acronyms and terminology

- SSB (to be not confused with "single-side-band" in radio transmission technology) means "Simple ultraSonic Binary", and is a file storage convention. The author uses it as a sort-of adjective applied to files.

- SSBF stays for "Simple ultraSonic Binary Format", used as a substantive.

So we can say that specific file "is SSB", while "this specification refers to SSBF". Occasionally the author might use a longer version of the latter by writing or saying something like "this file is encoded in SSB format".

Note from the author: _I don't like specially acronyms, but this one was quite a necessity, given its expansion being really very long and quite annoying; I hope "SSB" and "SSBF" are reasonably easy to remember, and am open to adopt any better idea. Of course I'm aware the acronym SSB overlaps another well-known object in communications engineering, and may then require some disambiguation, maybe a name change: I'm open to it, too_.

## Drawbacks of extant ultrasonic anemometer data formats

Data from three-dimensional ultrasonic anemometers are most often collected in proprietary format files, with typically a hourly organization.

Adoption of a hourly organization, i.e. files containing data collected during a specific hour, is quite a historical heritage: at 10 Hz, 36000 sonic quadruples (u,v,w,t) can be expected in a file: a number manageable enough, and likely to be stored in RAM on most computers from ate Nineties onwards.

The lack of standardization, however, made difficult to share data among users. The textual nature of some of these formats added to this issue, inducing inefficiency in data retrieval and storage.

The SSB format was developed to address these problems: it is a binary format, efficiently accessible in stream mode.

## Advantages of SSBF

Its _daily_ organization also encourages to deal with phenomena occurring on a time scale longer than one hour, as for example the tendency of airflow to recirculate in breeze regimens, which can be fully appreciated on a daily scale; addressing the study of such more-than-hourly phenomena using hourly files could be (by the author's direct experience) quite exposing to a lot of nitty-gritty code-writing to just arrange data so that hour 'i' really precedes 'i+1' in memory, time which could be used more productively addressing the scientific problem itself.

In addition, the size of an SSB file is "very small" for today's computers (somewhat less than 9 megabytes for data sampled at 10 Hz), and in the same time "large enough" to allow fast data transfer rates on local and network channels. Also, adopting daily files instead of hourly allow users to deal with 24 time less files, a strong bonus in operating systems placing penalties to large data file counts (as the author knows, Microsoft Windows _was_ prone to this problems in the past; and anyway, anyone who tried to list a some-years data set organized hourly on an FTP connection while counting time to gather children after school knows very well which the consequence of a "many extremely small files" approach means - a little cruelty, imposed by technological constraints in 1990s, but no longer needed in 2024).

## What's inside this repository

In this repository you can find two things:

- A _specification_ of SSBF format. This actually is the _main_ repository contents, and can find it further on in this document.

- An example, "reference" implementation of a library for importing, reading and writing SSB files.

The reference implementation, in directory "/src" of this repo, is written in Fortran 2008, assuming use of GNU Fortran or Intel Fortran compilers on a little-endian architecture. It could have been written in any other programming language providing access to binary data (e.g. C/C++, D, Julia, Python, ...), but the author choice was modern Fortran, due to its wide use in meteorology and fluid dynamics communities, and code simplicity to the eyes of non-professional programmers.

It is not excluded other (non-reference) implementations will be made available in future. And besides, would any of you want to contribute a version you can contribute to the code base, or fork the repository. The initiative is _open_, and participation is free, also thanks the permissive MIT license under which the code is distributed.

## Availability of data in SSB form

As of the publication date of this readme file, 23 February 2023, the author knows a significant sample of SSB data encoding is under way by the met unit of ARPA Lombardia for stations of their SHAKEUP micro-meteorological network, and will be available upon request as soon as the encoding is complete. The data set amounts in some tens of gigabytes.

## Standardization

Contacts with other institutions is in progress across Europe. Although not (yet) an official standard, effort is ongoing to allow an easy interchange of minimally-sized ultrasonic anemometer data among users. Would interest grow enough the author, who is actually an IEEE member, will evaluate whether to submit the SSBF as a standard proposal to the IEEE board. Your support (coding/documenting/testing if possible, but also emotionally is welcome) and sign of interest is essential in this way.

## SSB format file naming convention

The name of data files in SSBF cannot be assigned at will, but must conform to the specification YYYY-MM-DD.ssb, where YYYY represents the year, MM the month number, and DD the day number. The extension ".ssb" is nominal and mandatory.

Example:

2012-03-08.ssb

No restriction whatsoever applies the directory an SSB file is stored in: SSB files can be copied, moved, compressed, used, freely.

## Versions scope

This specification refers to SSB _version 1.0_ (limited to _ultrasonic anemometer quadruples_ each containing the three components of wind vector and sonic temperature) and _version 2.0_ (also supporting scalar values, and with a high-resolution, sub-second time stamp).

Versions 1 and 2 are not mutually compatible, and may be used for different use cases, version 1 privileging compactness and access speed, while version 2 is more general, a tad less efficient, and more time-granular.

## Version 1 format description

### Byte-endianness

SSB files are stored in _little-endian_ form.

This is the native byte endianness on Intel, AMD and ARM architectures. As far the author knows, some other widely diffused processors are big-endian (PowerPC among them). Interoperability has been tested between a Hewlett-Packard Z20 workstation (Intel Xeon) and an Apple MacBook Pro (M1 silicon, a 64-bit ARM incarnation), by direct file transfer and read.

If using a big-endian architecture, the "open" statement acting on SSB files should be changed to specify little-endian form explicitly. This should be possible as the author knows on various compilers, but actual mode and syntax may vary. A cruder alternative, that is reading the whole data byte-wise and then converting to the appropriate local byte endianness, is also possible - the author preferred not using it in her "reference" implementation in sake of code clarity.

### Data type abbreviations

By its very nature, a "binary" format is strongly types, and a necessity arises to describe it in as a programmng language independent possible way.

The following list shows the data types employed in Version 1 SSBF, plus likely extensions usable to document Version 2, would it exist on some time.

- I1: Signed integer, 1 byte.
- I2: Signed integer, 2 bytes, little-endian.
- I4: Signed integer, 4 bytes, little-endian.
- R4: Floating point number, IEEE-754 rev. 2008 format, 4 bytes, little-endian.
- R8: Floating point number, IEEE-754 rev. 2008 format, 8 bytes, little-endian.
- Cn, with 'n' a positive integer: fixed-length character string, ASCII, 1 byte per character, no encoding.

### Data sequence

The following list presents the contents, in order, of an SSB V1 file.

- C6, "sMagicSequence", should be "ssb_v0" (unchanged from the author's internal "Version 0"; _will change_ on version 2; will not change for subversions of version 1, and in case backward compatibility will be maintained with the possible exception of specifying the currently not specified "iReserved1" and "iReserved2").
- I1, "iReserved1", currently not specified for version 1, subversion 0; may be used in future subversions of version 1, and a definition will then be added.
- I1, "iReserved2", currently not specified for version 1, subversion 0; may be used in future subversions of version 1, and a definition will then be added.
- I2, "iYear", year of the date YYYY-MM-DD to which the data within file refer.
- I1, "iMonth", month of the date YYYY-MM-DD to which the data within file refer.
- I1, "iDay", day of the date YYYY-MM-DD to which the data within file refer.
- I4, "iNumData", total number of raw data in current file.
- I4(0:23), "ivNumData", array 0:23 of the total number of data in current file (may be 0 for 0 or more hours).
- I2(1:iNumData), "ivSecondStamp", array 1:iNumData) of seconds-bound time stamp (values from 0 to 3599).
- I2(1:iNumData), "ivU", X component of wind vector, in cm/s.
- I2(1:iNumData), "ivV", Y component of wind vector, in cm/s.
- I2(1:iNumData), "ivW", Z component of wind vector, in cm/s.
- I2(1:iNumData), "ivT", sonic temperature, in hundredths of °C.

Data is stored "column-wise", first all the X wind vector components, in order of increasing time stamp. Also notice that

  iNumData == ivNumData
   
and the starting position of hour 'i' in vectors "ivSecondStamp", "ivU", "ivV", "ivW", "ivT" is

  1 + sum(ivNumData(j), j = 1:(i-1)) if i > 1, or
  1 in case i = 1
  
The final position of hour 'i' in vectors is the initial position plus ivNumHours(i) minus 1.

Data in vectors are only valid: invalid data are not stored in SSB files.

Because of that, and the fact "ivSecondStamp" is integer, it normally happens some consecutive data items share the same stamp value. This is a desired feature for the kind of processing for which SSB V1 has been conceived, namely eddy covariance and the like. In fact, counting repetitions in second stamp is used on read to estimate the sonic sampling frequency. Seconds stamp repetitions also imply that time-stamp based averaging scheme cannot operate below 1s resolution. Higher-than-1s resolutions can be obtained through proper use of modular arithmetics (as of the "mod" operator in Fortran) - this approach is incidentally followed in the example application "averager", with the only restriction that the averaging time is an exact divisor of 3600s (ie 1 hour).

### Rationale behind column-wise format

Why column-wise? The reason has to do with the way memory circuitry is organized in current-technology random-access memories: contiguous cells can be retrieved in bursts, at a speed limited only by the memory raw bandwidth.

That given, the author did choose to store all data in the least-fragmented manner, that is, daily-columnwise.

In practical terms, this allows a somewhat "faster" access to data on read.

Data are retrieved from disks however, and advantages really become visible only once the data have been read from disk and reside in memory: compared to random-access memory all disks (including the solid state) are orders of magnitude slower. So in many cases the author expects only a marginal improvement during disk-intensive computing phases.

However: marginal is still better than nothing.

### Rationale behind I2 type for actual data

The decision to store vectors "ivU", "ivV", "ivW" and "ivT" as I2, that is signed 16 bits (2 byte) integers, has been taken after an evaluation of current ultrasonic anemometer technology. Measurement resolution is in the order of 1 cm/s (1 hundredth of °C for temperature), with extremal values in the order of +/-100 m/s for wind, -100 to +100 °C for temperature - actual limits are much smaller. This allows to encode wind and temperature data in "fixed point" with two decimal digits, occupying somewhat less than 16 bytes.

However, even employing techniques like delta modulation, a further compression to 8 bits is not always possible, and a huge increase in complexity would be necessary. So the author considered 16 bits as the best technical compromise between efficiency and simplicity.

## Version 2 format description

### Time granularity and file naming

Version 2 SSB files are always _hourly_. No daily packing, like in Version 1, is supported.

SSB version 2 files conform to the following naming convention:

_YYYY_-_MM_-_DD_._HH_.ssb

In this specification, _YYYY_ designates the year, _MM_ _DD_ and _HH_ the two-figures (with leading zeros) month, day and hour.

SSBF Version 2 is a format specification, so it strictly refers to file structure with no mention to directory organization. In real-world uses of SSBF, files are often stored to flat directories, or to subdirectories adhering to "Metek convention", with names

_YYYY_-_MM_

### Data sequence

The following list presents the contents, in order, of an SSB V1 file.

- C6, "sMagicSequence", should be "ssb_v2".
- I2, "iNumAnalog", number of analog data, with valid values from 0 (no analog data) to 10 (maximum allowed for Metek uSonic-3 and USA-1 anemometers.
- C16(0..iNumAnalog), "svAnalogName", names of analog data columns (none in case of no analog data).
- I2, "iYear", year of the date YYYY-MM-DD to which the data within file refer.
- I1, "iMonth", month of the date YYYY-MM-DD to which the data within file refer.
- I1, "iDay", day of the date YYYY-MM-DD to which the data within file refer.
- I1, "iHour", hour to which the data file refers.
- I4, "iNumData", total number of raw data in current file.
- R4(1:iNumData), "rvSecondStamp", array 1:iNumData) of within-hour time stamp (values from 0.0 to 3599.999).
- R4(1:iNumData), "ivU", X component of wind vector, in cm/s.
- R4(1:iNumData), "ivV", Y component of wind vector, in cm/s.
- R4(1:iNumData), "ivW", Z component of wind vector, in cm/s.
- R4(1:iNumData), "ivT", sonic temperature, in hundredths of °C.
- R4(1:iNumData, 1:iNumAnalog), "rmData", value of scalars, already converted to their own physical units).

Data is stored "column-wise", first all the X wind vector components, in order of increasing time stamp, and then all other columns in turn.

Data in vectors are only valid: invalid data are not stored in SSB files.

Time stamps of consecutive data items are allowed to have tha same time stamp; more generally, rvSecondStamp(i) <= rvSecondStamp(i+1) for all i in interval from 1 to n - 1.

## Reference implementation for Version 1

### Import formats

The actual import formats are not documented here in detail, and the interested reader may find them directly in the source of function "DailyGet". Coding is hopefully straightforward by the way.

The reason for this is the supported formats are very specific to the author's professional life. The only potentially standard is SonicLib, not supported in current implementation, and characterized by huge data volumes (almost 10 times the homologous binary form).

Anyway, here is the list of supported formats to date, in sake of completeness and to the ones interester:

- MeteoFlux Core V1
- MeteoFlux Core V2

### Modules


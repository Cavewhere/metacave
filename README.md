# metacave
## One cave survey data format to rule them all

(In progress!)

Metacave is a JSON cave survey data format.  That means it's:
* Way easier to parse and interpret than any other formats out there
* More human-readable and futureproof
* Extensible!
  * You can add all the metadata you want to any shot, station, trip, or cave,
    and give it a nice structure
  * You can embed program-specific features in it, without ruining another
    program's ability to load the base survey data
* Easy to store in MongoDB

## Specification
(go to [README.md source](README.md) to avoid horizontal scrolling)

### Distance Units

```
"in": inches
"ft": feet
"yd": yards
"m" : meters
"km": kilometers
"mi": miles
```

### Angle Units

```
"deg" : degrees
"min" : minutes  (1/60   degree)
"sec" : seconds  (1/60   minute)
"grad": gradians (1/400  circle)
"mil" : mils     (1/6400 circle)
```

### Additional Inclination Unit

```
"%"   : percent grade
```

### Structure

```
{
  "caves": {                                Required - this is where you define all            
                                            your survey measurements

    "Fisher Ridge": {                       Everything inside here is in Fisher Ridge cave

                                            If you connect to another cave, don't worry -
                                            we'll cover how to handle that below

      "fixedStations": [

        {                                   This is a fixed station group, whose only
                                            purpose is to specify the default units, UTM 
                                            zone and datum shared by all stations in it

          "distUnit": "m",                  Required - the default unit for all distances
                                            in this fixed station group

          "angleUnit": "deg",               Required - the default unit for all lat/lon
                                            angles in this fixed station group

          "utmZone": "16N"                  Optional - the UTM zone for all north/east
                                            locations
                                            Required if both north/east locations and
                                            lat/lon locations are used
                                            Only N/S are allowed, no MGRS letters

          "datum": "WGS84"                  Required - the reference ellipsoid for
                                            locations 
                                            Programs are not required to support anything
                                            other than WGS84

          "stations": {

            "A1": {                         This is the fixed position for station A1

                                            Either a "north"/"east" pair or a "lat"/"long"
                                            pair must be defined
                                            It is an error if both pairs are defined

              "north": 4645328,             Optional distance - northing from the UTM
                                            zone origin
                                            Default unit: "distUnit"

              "east": 502134,               Optional distance - easting from the UTM
                                            zone origin
                                            Default unit: "distUnit"

              "lat": 34                     Optional angle - latitude
                                            positive: north of equator
                                            negative: south of equator
                                            Default unit: "angleUnit"

              "long": -56                   Optional angle - longitude
                                            positive: east of prime meridian
                                            negative: west of prime meridian
                                            Default unit: "angleUnit"
  
              "elev": 345,                  Required distance - elevation
                                            positive: above sea level
                                            negative: below sea level
                                            Default unit: "distUnit"
            },

            "Q5": {                         This is the fixed position for station Q5
              ...
            },

            ...
          }

        },

        {                                   (Another fixed station group)
          ...
        },

        ...
      ],
      "trips": [
      
        {                                   This is the start of a trip

          "name": "Tricky Traverse",        Optional - a name for this trip
          
          "date": "1981-02-14T00:00Z",      Optional - the date of this trip, in ISO8601
                                            format
          
          "surveyors": {                    Optional - members of the survey team
            "Dan Crowl":   "sketch", 
            "Keith Ortiz": "frontsights", 
            "Chip Hopper": "backsights", 
            "Peter Quick": [
              "lead tape",                  You can specify multiple roles in an array
              "photos"                        
            ],
            "Larry Bean": null              Use null if you don't want to specify a role
          }

          "distUnit": "ft",                 Required - the default unit for all distances
                                            in this trip

          "angleUnit": "deg",               Required - the default unit for all angles in
                                            this trip
                                            Supported units:
                                            "deg" : degrees
                                            "grad": gradians
                                            "mil" : mils (1/6400 of a circle)

          "backsightsCorrected": false,     Required - true if backsights are corrected,
                                            false if not
          
          "azmFsUnit": "deg",               Optional - overrides default unit for
                                            frontsight azimuths

          "azmBsUnit": "grad",              Optional - overrides default unit for
                                            backsight azimuths
          
          "incFsUnit": "%",                 Optional - overrides default unit for
                                            frontsight inclinations
                                            Additional supported unit:
                                            "%": percent grade
          
          "incBsUnit": "mil",               Optional - overrides default unit for
                                            backsight inclinations
          
          "distCorrection": 1.5,            Optional distance - correction added to the
                                            distances.  Doesn't apply to LRUD/NSEW
                                            Default unit: "distUnit"
          
          "azmFsCorrection": 1,             Optional angle - correction added to the
                                            frontsight azimuths
                                            Default unit: "azmFsUnit", then "angleUnit"

          "azmBsCorrection": 2,             Optional angle - correction added to the
                                            backsight azimuths
                                            Default unit: "azmBsUnit", then "angleUnit"

          "incFsCorrection": 0,             Optional angle - correction added to the
                                            frontsight inclinations
                                            Default unit: "incFsUnit", then "angleUnit"
          
          "incBsCorrection": 0,             Optional angle - correction added to the
                                            backsight inclinations
                                            Default unit: "incFsUnit", then "angleUnit"
          
          "survey": [

            {                               This is a station.  The survey must begin and
                                            end with stations and have a shot in between
                                            each pair of stations
                                            Enter null where a station should be to turn
                                            the surrounding shots into splay shots

              "station": "A1",              Optional - the station name.  If omitted,
                                            turns the surrounding shots into splay shots
                                            (and all other fields here will be ignored)

              "cave": "Mammoth Cave",       Optional - specifies this station is in
                                            a different cave, for connections.  Stations
                                            in different caves can have the same name, but
                                            will be considered distinct.
              
              "isEntrance": true,           Optional - true if the station is an entrance,
                                            false (or omitted) if not
              
              "lrud": [5, 4, 0, 2],         Optional distances - the LRUDs at this 
                                            station.  They will also be associated with
                                            the surrounding shots
                                            Default unit: "distUnit"
              
              "lrudAzm": 3,                 Optional angle - the azimuth that is forward
                                            relative to the LRUDs.
                                            If omitted, the default is bisecting the
                                            previous and next shot azimuths
                                            (unless station is a dead end, then 
                                            perpendicular to shot)
                                            Default unit: "angleUnit"

              "nsew": [3, 2, 4, 6]          Optional distances - the NSEWs at this
                                            station. They will also be associated with
                                            the surrounding shots.
            },

            {                               This is a shot from the previous station to
                                            the following station
                                            Enter null where a shot should be to indicate
                                            there is not a shot between the surrounding
                                            stations (just like leaving a row of
                                            measurements blank in survey notes)

              "dist": 15.3                  Optional distance - the distance between the 
                                            surrounding stations.
                                            If omitted, means there was no shot between 
                                            the surrounding stations (just like leaving
                                            a row of measurements blank in survey notes)
                                            Default unit: "distUnit"

              "dist": "auto"                only allowed if the from and to
                                            station are fixed and no azimuths or
                                            inclinations are given; tells the
                                            program to draw passage connecting
                                            the stations anyway

              "fsAzm": 204.5                Optional angle - the frontsight azimuth
                                            Default unit: "fsAzmUnit", then "angleUnit"

              "bsAzm": 22.5                 Optional angle - the backsight azimuth
                                            Default unit: "bsAzmUnit", then "angleUnit"

              "fsInc": -5                   Optional angle - the frontsight inclination
                                            Default unit: "fsIncUnit", then "angleUnit"

              "bsInc": 6                    Optional angle or "up" or "down" - the
                                            backsight inclination
                                            Default unit: "bsIncUnit", then "angleUnit"
            }, 

            ...

            {                               The survey must end with a station
              "station": "END"
            }
          ]
        }
      ]
    } 
  }
}
```

### Individual unit overrides

Anywhere you put a quantity with default units, you can instead override the default unit
by entering an array like this:

  "dist": [4, "m"]                          distance is 4 meters even if "distUnit": "ft"

  "dist": [5, "ft", 7, "in"]                distance is 5 feet 7 inches

  "dist": [4, "in", 6, "ft", 2, "m"]        distance is 4 inches + 6 feet + 2 meters
                                            this is absurd, but programs will be simpler
                                            if they don't have to worry about weird order
                                            or combined unit systems

  "azmFsCorrection": [2, "grad"]            frontsight azimuth correction is 2 gradians

  "lat":  [23, "deg", 26, "min", 21, "sec"] 23° 26′ 21″ (the Tropic of Cancer)

  "lrud": [4, 5, 1, [5, "in"]]              left, right, and up are in default units,
                                            down is 5 inches

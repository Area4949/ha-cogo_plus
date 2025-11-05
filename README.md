# Home Assistant - Support for custom "zone" shapes
Home Assistant currently only has support for circular zones (as of November 2025). Many people have voiced the desire to have support for polygon zones. I have come up a solution that does not require a full integration package. Not only does this solution provide support for polygons, but also provides support for polylines (corridors), and since it was very simple to add, duplicated support for circles. Furthermore, it can easily be extended to other types of shapes.

## Custom Templates Macros
This solution to support multiple "zone" shapes is to utilize the 'Home Assistant - Reusing templates system' (https://www.home-assistant.io/docs/configuration/templating/#reusing-templates) to incorporate all the cogo math and support functions required.

## cogo_plus_macros.jinja
This is the support file that contains all the math functions to support the various zone shapes.
Shapes currently supported:
- Circle (Point with Radius)
- Polygon
- Polyline (corridor)

## Installation and usage
1) Copy the 'cogo_plus_macros.jinja' file to your home assistant config/custom_templates/ folder. Create this folder if it does not already exist. See the 'Home Assistant - Reusing templates system' documentation.
2) Edit your Configuration.yaml file to define your zones. See the provided configuration.yaml example that has all the currently supported zone shapes defined. Make sure you provide the same attributes as in the example, because robust error checking is currently not provided.
3) Re-start Home Assistant
4) Create an 'in zone' template sensor that loads, then calls the primary testing function. The primary testing function is named: 'device_in_zone'. In the following example, the sensor will have either a true or false state.

   eg:

           {% from 'cogo_plus_macros.jinja' import device_in_zone %}
           {% set pt1 = [ state_attr('device_tracker.an_iphone_3', 'longitude'), state_attr('device_tracker.an_iphone_3', 'latitude') ] %} 
           {{ device_in_zone(pt1, 'sensor.coastal_trail_zone_north') }}

## No 'Front End' is currently provided.
The current version of this 'add-on' does not have a front end to help in defining the zone shapes. See the "What is the easiest way to get a zone (shape) list of geodetic points into the format that this 'add-on' requires?" section under the FAQ.

## To-Do:
  - re-write applicable functions to use the 'do returns' feature? Jinja is really odd in that appears to sometimes change the data types on-the-fly on an exit. Had a heck of a time making everything work properly by forcing to_json and from_json...
  - add support for named devices(entities) that have tracker capability. Currently the tracker test point is built outside of the 'device_in_zone' function...
  - add and/or test support for dynamic zones..eg, a 'Zone - Radius' can have it's center point defined by the geo coordinates of a tracker device, or some points of a polygon can be...
  1) use case example: Motor home as the 'base', or an aircraft carrier, or a cruise ship. the zone shape would 'move' and 'rotate' with it as it moves accross the globe.
  2) use case example: A shared bounday between zones. move the coords of this boundary once, and it dynamically updates both..similar conceptionally to autocad 'snap'
  3) use case example: Set up a zone, based on multiple tracker devices. each device is a point of the polygon zone. Move any of them, and the polygon morphs with it.
    
  - add support for storing 'persons' in custom attributes - similar to  HASS Zones. Can do this via calling a macro function that sets the attrib info,,,
  - maybe add support for the 'gps_accuracy' attribute for phones, etc.
    
   -- this may be where we utilize the "Zone - Buffer" type and/or dynamically modify the "Zone - Polyline" offsets. ie a fuzzy boundary test???
    
  - Zone types to be added:
    
   -- Zone - Buffer
    
## Technical discussion and FAQ
### The importance of 'fuzzy equals' in cogo math functions
I have no idea whether standard libraries (such as Shapely for python) have built in fuzzy equals, but if they don't, then there is a VERY HIGH likelyhood that using these library functions as-is will return results that are not correct under certain conditions. The reason for this is because when comparing two real numbers (or two lists of real numbers, as in points), the two identical numbers can differ slightly if different methods are used to calculate them. So a rigid '==' test may return false, even though the two values are in fact 'the same'. This is why autolisp for AutoCad has included the "equal" function since inception. With this "equal" function, you can specify a fuzzy amount to compensate for the very slight difference that may result from the different methods of calculation.

Since I have based all of the jinja functions on my prior autolisp functions, the "fuzzy equal" tests are used where they are needed to ensure correct results are provided under all conditions. Jinja did not have an equivilent "equal" function, so I wrote one from scratch to provide the same functionality as the autolisp one.

### Why define the shapes in the 'Configuration.yaml' file and not in a GeoJSON file?
Because I want the user to have the greatest flexibility with the potential to incorporate 'dynamic' points in the zone shapes definitions. 

For example, one could define a 'Zone - Radius' with the center point defined with a jinja function. And this function could be a tracking device's current geodetic position. What use is this you may ask....Well one could define a 'Home' zone based on a motorhome's tracking sensor. So, the 'Home' zone would be dynamically defined and centered on wherever the motorhome is located at any particular time.

Or, perhaps we want to define a polygon zone with common points to an adjacent polygon zone. The common points in the second zone can be defined by the points in the first one via jinja list parsing functions. Move the common points of the first zone, and the points of the second zone dynamically update.

Or, perhaps we want to define a polygon zone with a set of tracking devices. Each point of the polygon can be defined by the tracking devices current geodetic position. This would allow for a fully dynamic zone. Move any or all of the individual trackers, and the zone dynamically updates.

### What is the easiest way to get a zone (shape) list of geodetic points into the format that this 'add-on' requires?
I recommend using Google Earth to define the geometry of the zones, then using the 'Save Place As' tool to export to a .kml file. 

Once a kml file has been created, use a text editor to open the .kml file, copy the "coordinates" section list and paste it into your favorite AI tool and ask that tool to convert the coodinates into the proper format. 

I use Perplexity and my prompts are (example) 

          modify this list to match the format of a list of points without the z value. -149.9714801762212,61.18220193377709,0 -149.9716773748038,61.18195478028175,0 -149.9716855038654,61.18188228574225,0 -149.9713782633992,61.1818286392424,0 -149.9713094444549,61.18183260821363,0 -149.9710822363566,61.18213539980892,0 -149.9714801762212,61.18220193377709,0

then

          modify to match this format:
          p_1:  "[-150.0473937469993, 61.16639584745547]"
          p_2:  "[-149.9701612614392, 61.16683672889118]"
          p_3:  "[-149.9637548431457, 61.16877164284922]"
          p_4:  "[-149.9447116679241, 61.16894553112301]"

This will get it into the correct format if you use perplexity. I'm sure other AI tools can do this with similar prompts...

         Here is your coordinate list reformatted to match the requested style. Each entry is labeled with p_n and formatted as:
         p_n: "[longitude, latitude]"
         Where n starts at 1 and increases consecutively:
         p_1: "[-149.9714801762212, 61.18220193377709]"
         p_2: "[-149.9716773748038, 61.18195478028175]"
         p_3: "[-149.9716855038654, 61.18188228574225]"
         p_4: "[-149.9713782633992, 61.1818286392424]"
         p_5: "[-149.9713094444549, 61.18183260821363]"
         p_6: "[-149.9710822363566, 61.18213539980892]"
         p_7: "[-149.9714801762212, 61.18220193377709]"

### What is a Polyline Zone, and what would I use it for?
Think of a "Zone - Polyline" as a pathway or roadway with a defined width. The width is defined in this 'add-on' as an "Offset" attribute. The "Offset" value is 1/2 the full width of the zone. This zone shape can be useful for knowing if a tracking device is inside or outside of a linear feature. 

### Can I combine zones if I want to?
Sure! If you define the 'in_zone' boolean template sensors as shown in the example, then you can further define and/or test for zone unions or zone intersections via the bolean 'and' and 'or' tests. eg: {{ device_in_zone(pt1, 'sensor.ak_airmans_zone_polygon') or device_in_zone(pt1, 'sensor.ted_stevens_airport_zone') }}

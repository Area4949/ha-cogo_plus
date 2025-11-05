# Home Assistant - Support for custom "zones" shapes
Home Assistant currently only has support for circular zones (as of November 2025). Many people have voiced the desire to have support for polygon zones. I have come up a solution that provides a solution that does not require a full integration package. Not only does this solution provide support for polygons, but also provides support for polylines (corridors), and since it was very simple to add, duplicated support for circles.

# Custom Templates Macros
This solution to support multiple "zone" shapes is to utilize the Home Assistant - Reusing templates system (https://www.home-assistant.io/docs/configuration/templating/#reusing-templates)

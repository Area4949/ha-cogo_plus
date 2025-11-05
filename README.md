# Home Assistant - Support for custom "zones" shapes
Home Assistant currently only has support for circular zones (as of November 2025). Many people have voiced the desire to have support for polygon zones. I have come up a solution that does not require a full integration package. Not only does this solution provide support for polygons, but also provides support for polylines (corridors), and since it was very simple to add, duplicated support for circles.

## Custom Templates Macros
This solution to support multiple "zone" shapes is to utilize the Home Assistant - Reusing templates system (https://www.home-assistant.io/docs/configuration/templating/#reusing-templates).

## cogo_plus_macros.jinja
This is the support file that contains all the math functions to support the various zone shapes.
Shapes currently supported:
- Circle
- Polygon
- Polyline (corridor)

## Technical discussion on cogo math and the potential pitfalls
### The importance of 'fuzzy equals' in cogo math
I have no idea whether standard libraries (such as Shapely for python) have built in fuzzy equals, but if they don't, then there is a VERY HIGH likelyhood that using these library functions as-is will return results that are not correct. The reason for this is because when comparing two real numbers (or two lists of real numbers, as in points), the two identical numbers can differ slightly if different methods are used to calculate them. So a rigid '==' test may return false, even though the two values are in fact 'the same'. This is why autolisp for AutoCad has included the "equal" function since inception. With this "equal" function, you can specify a fuzzy amount to compensate for the very slight difference that may result from the different methods of calculation.


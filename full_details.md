#Syntax details:

1. Introduce new "geometry" keyword for layer collections. Can be applied to class and id selectors and can receive their own styling defaults. They are special key words like "zoom".

        WHY: Different rendering assumptions about styles applied to different geometries is unclear, not future-proof/predictable. The terms icon/marker/point are confusing between languages. But actually visualizing point data should be easy for new users.
    
        * Point
        * Line
        * Polygon
        * Grid/Interaction
        
            EXAMPLES IN USE: 
                #layer line {}
                .class line {}
            
        Note: All these technically refer to single- or multi- collections. If you need to grab one in a multi-, there are now selectors for that specifically.

2. Default symbolizers (avoid confusing terms like markers and points):
    
        * stroke-width
        * stroke-color
        * stroke-opacity
        * stroke-dash-array
        * fill-color
        * fill-opacity
        * anchor/marker/point ack, see below
        
        The examples:
        fill: url(...) #9f0 1.0;
            expands to:
                fill-image: url(...);
                fill-color: #9f0;
                fill-opacity: 1.0;
            placement w/respect to geometry:
                polygon inside
        stroke: url(...) #9f0 repeat 20.0;
            expands to:
                stroke-image: url(...);
                stroke-color: #9f0;
                repeat: 20.0;
            placement w/respect to geometry:
                polygon edge
                line edge
        ***mark(er)***: url(...)
            what do we call this fracking thing?
                marker < good for GMaps people, kinda Mapnik, too
                dot < Ha, Stamen default
                icon < Windows and Mac finder/explorer
                symbol < good for Illustrator and Flash and SVG people
                spot
                anchor < more html
            expands to:
                fill: #f90;
                fill: url(...);
            default placement w/respect to geometry:
                marker-placement (anchor)
                polygons - centroid/onsurface
                lines - centroid/onsurface
                points - centroid/onsurface
            awesome placement w/respect to geometry:
                vertex:first
                vertex:last
                vertext (implied all)
                rings of polys?
        ??????        
        -------- anchor-image _(was point-image)_
        ---------------- anchor-width _(was point-width)_
        ---------------- anchor-color _(was point-color)_
        ---------------- anchor-opacity _(was point-opacity)_
        ---------------- anchor-placement _(was point-placement)_
        ---------------- DO THESE REPEAT FOR stroke and fill images???
        -------- stroke-image
        -------- fill-image
        -------- annotation/text???
        ---------------- shield is a child???
        -------- NOTE: Want another stroke or fill, etc? Use an arbitrary attribute.
        -------- DEPRECIATE (in syntax): 
        ---------------- marker
        ---------------- polygon-fill
        ---------------- line-pattern
        ---------------- polygon-pattern
        ---------------- shield
        ---------------- QUESTION: What about buildings? Ignore for now?
        -------- QUESTION:
        ---------------- can I give any "color" attribute RGBA, gradient, or urls to png, svg, etc? Why do I need separate "image" bits?
    
3. Complex styles can be viewed in an expanded form.

4. Allow remote reference to stylesheet bits (add import directive).

5. @variables for Appearance stuff

        * Color swatches (rbga)
        * Graphic styles (per geometry type?)
        * Text character styles

6. Named color swatch groups: from http://colorbrewer2.org/
        
        * Classification steps: 3 to 12 steps; continuous
        * Data type: sequential, diverging, qualitative
        * Color scheme: multi-hue, single-hue

7. Gradients

        * Are like Named color swatch groups, just with continuous tone between the stops.
        * Tied to absolute or relative data values.
        * If relative %s, might have a cached version that uses the actual data values for the dataset it's applied against.
        * Can symbolize raster grids, too. Or symbolize geometries versus data.

8. Data statistic keywords:

        * MIN, MAX, MEAN
        * Others?

9. The Less attachements and filtering, etc

10. When something is defined, what is the default options. When they are defined. When they need to be updated.


# Implementation:

1. Implemented first in Mapnik.
2. Abstract enough that is can easily be implementable in other applications. Examples: Google Maps API, Google Earth (kml), ArcMap, MapServer, GeoServer, CartoDB.
3. Allow for streaming vectors.
4. Think about server versus client. Wither the browser (ever powerful and resurgent).


# Avoids:

1. Callbacks like f(eval) --- from Mike


# DEMOS given:

* high roads
* OSM styles


# Open questions:

1. What doesn't work in the browser

        * end of thing, or end of clipped bit (for line caps)
        * stats
        * layering/data-ordering:
                       for data you don't control
                       especially for sorting city townspots, labels
                       YES: primitive like Order by ASC/DESC like SQL
                       YES: adding something to the Layer def properties???
                       NO: z-index as an expression (for a feature per layer, or across layers?)
        * label placements (ref. Tom Carden)
        * inability to have all geodata in the DOM, streaming data, real time data
                       browser implementation knows that min/max aren't available. Therefore the stylesheet becomes invalid. Does it warn and draw or just fail. Or option to go get min/max?
                    also for streamed vector data when it is streamed by tile to the browser...
                        where are the tile edges? (masking is the key)
        * remixing CSS on the fly
        * not sure where <Layer group="xyz"> syntax went. Mike??
        * generalized geometries, streamed geometries, data topologies (typologies)
        * 80% common data: eg "roads" - data hints on source data, then styling (auto?)
        * how do you prefix the (syntax) things that aren't core?
                       especially relevant for sharing, to be easy for the end user and for helping experts troubleshoot
                       YES core is not prefixed
                       YES awesome (experimental) is
                       where is that line drawn (har har)
                       helps with validation, warnings, errors, optimizations per app implementation.
                       minimal grammar for carto (repository that doesn't have a ref. implmentation). with no properties.
                       then ...
        * StyleHub proposal - stubbed out later here: https://github.com/nvkelso/geo-how-to/wiki/Style-Hub
                       need to copy-paste and mostly work.
                       shoul just work for "80% common" data
                       should be able to use's design:
                            @import "style.mss"; 
                            and refernce styles in there in your main stylesheet. same as css.
                       if it doesn't work, the syntax should make it more obvious
                       polygon-fill: chloropleth( bins=8, [POPULATION], colors)
                           which expands/cached? to something like
                       popuation-fill: "population" 10 #f00 100 #f90 1000 #ff0
                       and the 10, 100, 1000 could be % instead?
                       ¿ Question about CSS gradient syntax??? Seems similar?
                       ¿ What is the FusionTable syntax???
                       KEY: Differnce: data exploration (data discovery) or final data presentation/publishing ;)
                               Kinda need a python thing to call that points at data source and stubs out a quick CSS of that with random colors (or using color paramater). NVK MM Tom
                               Kinda like the facet tool.
                        use of @ variables in
                            .roads( @classified ) {
                                stroke-width: @classified;
                            }
                            #roads( [FIELD] ) {
                                .polygon,
                                #lakes {
                                    set tag = blah;
                                    set eleemnts = .polygon;
                                    }
                                }
                            }
                        
2. How is data statistics (metadata) stored (cached)?
    
        * MIN, MAX, MEAN, STDDEV, others?
        * similar to CSS conditionals (fallbacks) with cached hints of those values?
        * This needs to be a parellel spec?

3. How is data statistics (metadata) calculated, updated? How would I ask the server for this? An external local application? How does this fail? How does it recover?
    
        * Using all the data? Part of the data? 
        * In view or arbitrary bbox?
        * If my file is less than X file size, do this quick thing. Else that slow thing.
        * Extent hard coded???

4. Are "min" styles simply an expanded form of normal/complex styles?

5. Are "min" styles a recommendation for showing complex rendering with a simple fallback version?

6. What is the "awesome" syntax for saying: "take this data, classify it into 4 buckets using natural breaks, and color using these colorbrewer.org swatches". D3 from Bostok seems to be good at this.

        * Partially implemented: https://github.com/nvkelso/thematic-carto-tools
        
        * EASY: polygon-fill: [population] 10 f00 100 #f90 1000 #ff0;
                    but can I use % instead of 10, 100, 1000?
                    but can I say MIN, MAX, etc?
        * HARD: polygon-fill: choropleth([POPULATION], bins=5, natural-breaks, @colors);
                    need a tool that turns this into the easy bit, parellel project to this?
                    @colors {
                        @color1: #000;
                        @color2: #111;
                        @color3: #222;
                        @color4: #333;
                        @color5: #444;
                    }
                    cached as??
                    refresh: [interval]
                        date/time
                        auto
                        none
                        daily, weekly, montly, annual
        * Appearance style mix-ins:
        * Use Less.js options for preprocessing
                 TIP: Carto should support this, but it's broken right now.
                    @import "roads.mss" <<< someone else's road styling
                                which has
                                .roads { stroke... }
                    #my_roads{ .roads; }
                    parameters
                                   .roads( @classified) {
                                       stroke-width: @classified;
                                    }
                                    #my-roads {
                                        .roads( [MY_CLASS] );
                                    }
                        option 2:
                                  mapCSS huh?
                        option 3:
                                  use of polygon, line geometry bits in styling. Migurski!!!
        * Data side:
            <Layer class="roads">
            <LayerGroup class="roads">
                                   <layer name ="highway3"></layer>
                        </LayerGroup>

7. The "Map" object needs to be made less special.

        * Should treat more as a tile0 polygon to style
        * Should carry just projection data, units, and the like

8. Normalize new TextSymbolizer syntax?

9. Are arbitrary attributes iterable only over the layer or over features? (Mike, add more here)

        * z-order is only modifiable for objects within a data layer.

10. How is display:none implemented. Let's see some examples for layers, geometries, symbolizers, class, id selectors.
        
        https://github.com/mapnik/mapnik/issues/925

11. Legends (haha)

12. Page layout for print (haha)

13. Webkit ideal for debug environment (Dane)

14. Use of : for polygon:hover. Use of :: for arbitrary attributes. What else?

15. Is it legal to say: stroke: "1px red";???

16. Can I give any "color" attribute RGBA (no, just RGB, A is taken care of with opacity key) gradient, or urls to png, svg, etc? Why do I need separate "image" bits?

17. Shield syntax:
        
        * ¿¿ Townspot { point-width: 10; point-color: red; text-on: point; text-dx: 10; }
        * ¿¿ Shield { stroke-image: url(shield.png) 100; text-on:stroke-image; text-face-name: Deja Vu; text-name: ref; }

18. Bindings/Pairings/Overflow/Depends-on
    
        .thing { 
              stroke-image: url(shield.png);
              stroke & {
                    text-name: [bla];
              }
        }
        
        .thing.successor {
            stroke & {
                text-face-name: Deja Vu;
            }
        }
        
        .thing {
            point-image: url(city.png);
            point -> {
                text-name: [bla];
            }
        }

###Housekeeping:

1. Setup a repository for our code.
2. Setup a wiki & blog for our documentation.

###Will this work? Let's truth squad and ask:

* Tab Atkins, G
* Shawn Allen, Stamen
* TMCW, DevSeed
* Brenden, G?
* Mike Bostok, D3 land
* Darfa, MapCSS, Kofik? < draw shields dynamically
* Gmps engineers


##Quotes:

* "Mmmm, smells like an attractive nuisance!" --Mike
* "In the future, we will have hover" --Dane?
* "&! (and not), outrageous" --Mike?



# Is this Friday notes now?
========

* https://github.com/mapnik/mapnik/issues/794
* https://github.com/mapnik/carto-spec/issues/3


# Carto basic show/hiding logic

* per layer (already in mapnik, and in the data file, needs to be in the style file, basically turns all symbolizers for that layer off)
* per symbolizer display-none: on/off

problem with showing/hiding problem:
    instead of switching syntax to point-auto: true/false;

Should have reasonable SYMBOL representation defaults, below.


###Geometry abstracts:

* point
* line
* polygon (multipolygon, etc)
* raster (grid)


#Graphic representations:

    point {
        width: 3;
    }
    
    #layer {
        point {
            display: none;
        }
    }
    
    .subthing {
        point: {
            stroke: 2;
        }
    }


#Carto 1.x Nitpicks

1. Formalize:

        when we're looking at FIELDS, we always use
            `[FIELD]`
        
        if you need to escape
            `/[foo\]`
        
        This is easy to remember and 
        
        Note: When a property expects a string, you need to wrap "[FIELD]", but not necc. @var.
        
        We want to insist on [FIELD] because it makes syntax highlighting easier (and thus code easier to write and to interpret when rendering).
        
        no way to know if [FIELD] is real or not unless at runtime. But that's okay.


2. What needs to be expanded?

        * variables get replaced
        * inheretance gets replaced
        
        Can't really do unquoted variables -- [FIELD] versus "[FIELD]" -- unless Mapnik supports natively.


3. Woops, that doesn't exist:

        When [FIELD] isn't valid, what to do?
        
        * warnings (silent or obvious), or treat as errors and fail completely as error
        * html/css will make it look funny if it's not right, but it'll kinda work (we like this)
        * return empty string or undefined or?? now it throws an error and won't run at all
        
        Why is this important: sharing styles between similar datasets should work (now they don't at all).


4. Mapnik variable filters

        (more powerful than CSS before, after pseudo selectors)
            `#rule[THING]>value]`
        
        same as
            `#rule[[THING]]>value]`
        
        same as
            `#rule[[THING11]]>[THING2]]`
        
        [] are used both for FIELDS and doing evals?
        
        "Ghostly manifestations of things that are in Mapnik"


###KEY (concept? likely NVK)

* How do you make a choropleth map?
* Can I say this data, these colors, with quantiles/naturalbreaks


#Back to basic syntax:

###Operations:

= >= <= < > !=


###Document this:

* Concatenate is a + not &

but

* "Name: [FIRST] [MIDDLE] [LAST]" is also valid, but doesn't work now. But it should.
* "Name: [FIRST + ',' + FIELD2]" Ack, is that valid?


###Magic constant:

* NULL
* TRUE
* FALSE

Example: Empty strings, NULL in number columns for Shps, etc.

Question: Does Mapnik have other types of Nulls? Typecasting magic by Mapnik now... ack!

So make sure Mapnik is being a good citizen, then tell others about that.


###Expression tokens:

tk tk tk


###Filters:

    5 classes, what are the breaks, what are the colors.
        #rule[ A="5" ][ B>= 7 ]
    
    same as:
        #rule[ A="5" ],
        #rule[ B>= 7]
    
    not liking:
    (it's better to do that type of stuff on the SQL bits)
        #rule[ A="5" OR B>= 7 ]
    
    not liking:
    (it's better to do that type of stuff on the SQL bits)
        #rule[ [A]="5" OR [B]>= 7 ]


###Filters extra:

Okay to compare:

* variable to variable, variable to [FIELD], variable to constant, variable to "", variable to #
* [FIELD] to [FIELD]
* constant to constant

Note: If the property expects a string, you need to wrap the thing in the string by saying "[FIELD]".


###Special filter:

Now: 
    [zoom=1]

if it's a FIELD not a special word:
    [[ZOOM] = 1]
or
    [zoom() = 1]

What are the special keywords for DATA STATS:
    `Dane can fill us in?`

Need to make explicate and document: "What would Dane do?" @aaronofmontreal


###Variables are good:

They accomplish stylesheet type of things. Examples:

    @color: #xxx;
    @textStyle {
        text-color: #yyy;
        text-fixe: 12;
    }


###What to do with variable side effects.

* The user can change the variable definition 1/2 way thru the stylesheet. This is weird, but okay.

* The implementation should: Warn on variable overriding. But still compile. Also warm if undefined state.

* App logic for merging reusabel styles: recommendation to error detect on merge and ask if append with new name, merge with exhisting variable definition.

> https://github.com/mapbox/carto/issues/116


#Questions:

* What can we do without Mapnik in Carto. 
* What is dependent on Mapnik3?
* Staging repos.

* Is the goal of to make Mapnik only read Carto+ or to have a built in interpretor of it that translates to native Mapnik XML? The C++ parser can do it fine. It's only the XML bit that's complicated? It is really nice to see what the finished stylesheet is (kinda the "min" expanded normal/awesome mode).

* Yes, XML should still be okay for MML files (says Dane).

###FRIDAY todos:

* Granularity of the syntax
* Edge case: Don't have all the data
* Data stats: MIN, MAX, etc.
* Browser specific bits client side (canvas w/ hardware acceleration seems to be the future, svg was hard, but we're kinda there)
* How do we keep it in the 20% time: 
* Compliant with simple version, but can hop up (to attachments, etc).
* MapGL... no public API. How might that be styled.
* FusionTables are stylable, but not fun.
* Maps API is stylable, but not fun.
* Default map styling for classes (blue rivers).

Styling KML is not fun. Earth implements as the opposite. Each feature looks for style.
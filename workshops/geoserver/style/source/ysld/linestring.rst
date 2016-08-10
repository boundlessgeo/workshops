.. _style.line:

Lines
=====

We will start our tour of YSLD styling by looking at the representation of lines.

.. figure:: /style/img/LineSymbology.svg
   
   LineString Geometry

Review of line symbology:

* Lines can be used to represent either abstract concepts with length but not width such as networks and boundaries, or long thin features with a didth that is too smallt o represent on the map. This means that **the visual width of line symbols do not normally change depending on scale.**

* Lines are recorded as LineStrings or Curves depending on the geometry model used.

* SLD uses a **LineSymbolizer** to record how the shape of a line is drawn. The primary characteristic documented is the **Stroke** used to draw each segment between vertices.

* Labeling of line work is anchored to the mid-point of the line. GeoServer provides a vendor option to allow label rotation aligned with line segments.

For our exercises we are going to be using simple YSLD documents, often consisting of a single rule, in order to focus on the properties used for line symbology.

Each exercise makes use of the ``ne:roads`` layer.

Reference:

* :manual:`Line Symbology <extensions/css/properties.html#line-symbology>` (User Manual | CSS Property Listing)
* :manual:`Lines <extensions/css/cookbook/line.html>` (User Manual | CSS Cookbook)
* :manual:`LineString <styling/sld-reference/linesymbolizer.html>` (User Manual | SLD Reference )

Line
----

A line symbolizer is represented by a :kbd:`line` key.  You can make a completely default symbolizer by giving it an empty map

.. code-block:: yaml
   
        line: {}

.. figure:: /style/img/LineStringStroke.svg
   
   Basic Stroke Properties



#. Navigate to the **Styles** page.

#. Click :guilabel:`Add a new style` and choose the following:

   .. list-table:: 
      :widths: 30 70
      :stub-columns: 1

      * - New style name:
        - :kbd:`line_example`
      * - Workspace for new layer:
        - Leave blank
      * - Format:
        - :kbd:`YSLD`

#. Fill in the style editor 

   .. code-block:: yaml
   
        line: {}

#. Click :guilabel:`Submit` 

#. Click return to the **Styles** page and click :guilabel:`line_example` 

#. Click :guilabel:`Layer Preview` to see your new style applied to a layer.
   
   You can use this tab to follow along as the style is edited, it will refresh each time :guilabel:`Submit` is pressed.

   .. image:: /style/img/line.png

#. You can see the equivalent SLD by requesting :kbd:`http://localhost:8080/geoserver/rest/styles/line_example.sld?pretty=true` which will currently show the default line symbolizer we created.

   .. code-block:: xml

      <?xml version="1.0" encoding="UTF-8"?><sld:StyledLayerDescriptor xmlns="http://www.opengis.net/sld" xmlns:sld="http://www.opengis.net/sld" xmlns:gml="http://www.opengis.net/gml" xmlns:ogc="http://www.opengis.net/ogc" version="1.0.0">
        <sld:NamedLayer>
          <sld:Name>line_example</sld:Name>
          <sld:UserStyle>
            <sld:Name>line_example</sld:Name>
            <sld:FeatureTypeStyle>
              <sld:Name>name</sld:Name>
              <sld:Rule>
                <sld:LineSymbolizer/>
              </sld:Rule>
            </sld:FeatureTypeStyle>
          </sld:UserStyle>
        </sld:NamedLayer>
      </sld:StyledLayerDescriptor>

We only specified the line symbolizer, so all of the boilerplate arround was generated for us.

#. Additional properties cane be used fine-tune appearance. Use **stroke-color** to specify the colour andwidth of the line.

   .. code-block:: yaml
      :emphasize-lines: 2
   
      line:
        stroke-color: blue

#. **stroke-width** lets us make the line wider

   .. code-block:: yaml
      :emphasize-lines: 3
   
      line:
        stroke-color: blue
	stroke-width: 2px

#. **stroke-dasharray** applies a dot dash pattern.

   .. code-block:: yaml
      :emphasize-lines: 4
      
      line:
        stroke-color: blue
	stroke-width: 2px
        stroke-dasharray: 5 2

#. Check the :guilabel:`Map` tab to preview the result.

   .. image:: /style/img/line_stroke.png

.. note:: The GeoServer rendering engine is quite sophisticated and allows the use of units of measure (such as :kbd:`m` or :kbd:`ft`). While we are using pixels in this example, real world units will be converted using the current scale, allowing for lines that change width with the scale.

Multiple Symbolizers
--------------------

Providing two strokes is often used to provide a contrasting edge (called casing) to thick lines.  This can be created using two symbolizers.

.. figure:: /style/img/LineStringZOrder.svg


#. Start by filling in a bit of boilerplate that we'll be using

   .. code-block:: yaml

      feature-styles:
      - rules:
        - symbolizers:
          - line:
              stroke-color: '#8080E6'
              stroke-width: 3px

   The line symbolizer is inside a rule, which is inzide a feature style.

#. Add a second symbolizer to the rule

   .. code-block:: yaml
      :emphasize-lines: 4,5,6

      feature-styles:
      - rules:
        - symbolizers:
          - line:
              stroke-color: black
              stroke-width: 5px
          - line:
              stroke-color: '#8080E6'
              stroke-width: 3px

   The wider black line is first so it is drawn first, then the thinner blue line drawn second and so over top of the black line.  This is called the painter's algorithm.

#. If you look carefully you can see a problem with our initial attempt. The junctions of each line show that the casing outlines each line individually, making the lines appear randomly overlapped. Ideally we would like to control this process, only making use of this effect for overpasses.

   .. image:: /style/img/line_zorder_1.png

   This is becaue the black and blue symbolizers are being drawn on a feature by feature basis.  For nice line casing, we want all of the black symbols, and then all of the blue symbols.

#. Create a new feature style and move the second symbolizer there.

   .. code-block:: yaml
      :emphasize-lines: 2,3,4,5,6

      feature-styles:
      - rules:
        - symbolizers:
          - line:
              stroke-color: black
              stroke-width: 5px
      - rules:
        - symbolizers:
          - line:
              stroke-color: '#8080E6'
              stroke-width: 3px

   Again we are using painter's algorithm order: the first feature style is drawn first then the second so the the second is drawn on top of the first.  The difference is that for each feature style, all of the features are drawn before the next feature style is drawn.

#. If you look carefully you can see the difference. 

   .. image:: /style/img/line_zorder_2.png

#. By using **feature styles** we have been able to simulate line casing. 

   .. image:: /style/img/line_zorder_3.png

Label
-----

Our next example is significant as it introduces the how text labels are generated.

.. figure:: /style/img/LineStringLabel.svg
   
   Use of Label Property

This is also our first example making use of a dynamic style (where the value of a property is defined by an attribute from your data).

#. To enable LineString labeling we will need to use the key properties for both **stroke** and **label**.

   Update ``line_example`` with the following:
   
   .. code-block:: css
      :emphasize-lines: 2,3

      * {
        stroke: blue;
        label: [name];
      }

#. The SLD standard documents the default label position for each kind of Geometry. For LineStrings the initial label is positioned on the midway point of the line.

   .. image:: /style/img/line_label_1.png

#. We have used an expression to calculate a property value for label. The **label** property is generated dynamically from the :kbd:`name` attribute. Expressions are supplied within square brackets, making use of Constraint Query Language (CQL) syntax. 

   .. code-block:: css
      :emphasize-lines: 3

      * {
        stroke: blue;
        label: [name];
      }

#. Additional properties can be supplied to fine-tune label presentation:
   
   .. code-block:: css
      
      * {
        stroke: blue;
        label: [name];
        font-fill: black;
        label-offset: 7px;
      }

#. The **font-fill** property is set to :kbd:`black` provides the label color.

   .. code-block:: css
      :emphasize-lines: 4
      
      * {
        stroke: blue;
        label: [name];
        font-fill: black;
        label-offset: 7px;
      }
      
#. The **label-offset** property is used to adjust the starting position used for labeling.
   
   Normally the displacement offset is supplied using two numbers (allowing an x and y offset from the the midway point used for LineString labeling).

   When labeling a LineString there is a special twist: by specifying a single number for **label-offset** we can ask the rendering engine to position our label a set distance away from the LineString. 
  
   .. code-block:: css
      :emphasize-lines: 5
      
      * {
        stroke: blue;
        label: [name];
        font-fill: black;
        label-offset: 7px;
      }

#. When used in this manner the rotation of the label will be adjusted automatically to match the LineString.

   .. image:: /style/img/line_label_2.png

How Labeling Works
------------------

The rendering engine collects all the generated labels during the rendering of each layer. Then, during labeling, the engine sorts through the labels performing collision avoidance (to prevent labels overlapping). Finally the rendering engine draws the labels on top of the map. Even with collision avoidance you can spot areas where labels are so closely spaced that the result is hard to read.

The parameters provided by SLD are general purpose and should be compatible with any rendering engine.

To take greater control over the GeoServer rendering engine we can use "vendor specific" parameters. These hints are used specifically for the GeoServer rendering engine and will be ignored by other systems. The GeoServer rendering engine marks each vendor specific parameter with the prefix **-gt-**.

#. The ability to take control of the labeling process is exactly the kind of hint a vendor specific parameter is intended for.
    
   Update ``line_example`` with the following:

   .. code-block:: css

      * {
        stroke: blue;
        label: [name];
        font-fill: black;
        label-offset: 7px;
        -gt-label-padding: 10;
      }

#. The parameter **-gt-label-padding** provides additional space around our label for use in collision avoidance.

   .. code-block:: css
      :emphasize-lines: 6
   
      * {
        stroke: blue;
        label: [name];
        font-fill: black;
        label-offset: 7px;
        -gt-label-padding: 10;
      }

#. Each label is now separated from its neighbor, improving legibility.

   .. image:: /style/img/line_label_3.png

Scale
-----

This section explores the use of attribute selectors and the :kbd:`@scale` selector together to simplify the road dataset for display.

#. Replace the `line_example` CSS definition with:

   .. code-block:: css

      [scalerank < 4] {
        stroke: black;
      }

#. And use the :guilabel:`Map` tab to preview the result.

   .. image:: /style/img/line_04_scalerank.png

#. The **scalerank** attribute is provided by the Natural Earth dataset to allow control of the level of detail based on scale. Our selector short-listed all content with scalerank 4 or lower, providing a nice quick preview when we are zoomed out.

#. In addition to testing feature attributes, selectors can also be used to check the state of the rendering engine.

   Replace your CSS with the following:

   .. code-block:: css

      [@scale > 35000000] {
         stroke: black;
      }
      [@scale < 35000000] {
         stroke: blue;
      }

#. As you adjust the scale in the :guilabel:`Map` preview (using the mouse scroll wheel) the color will change between black and blue. You can read the current scale in the bottom right corner, and the legend will change to reflect the current style.

   .. image:: /style/img/line_05_scale.png

#. Putting these two ideas together allows control of level detail based on scale:

   .. code-block:: css

      [@scale < 9000000] [scalerank > 7] {
        stroke: #888888;
        stroke-width: 2;
      }
      [@scale > 9000000] [@scale < 17000000] [scalerank < 7] {
        stroke: #777777;
      }
      [@scale > 1700000] [@scale < 35000000] [scalerank < 6] {
        stroke: #444444;
      }
      [@scale > 3500000] [@scale < 70000000] [scalerank < 5] {
        stroke: #000055;
      }
      [@scale > 70000000] [scalerank < 4] {
        stroke: black;
      }

#. As shown above selectors can be combined in the same rule:

   * Selectors separated by whitespace are combined CQL Filter AND
   * Selectors separated by a comma are combined using CQL Filter OR

   Our first rule `[@scale < 9000000] [scalerank > 7]` checks that the scale is less than 9M AND scalerank is greater than 7.

   .. image:: /style/img/line_06_adjust.png
   

Bonus
-----

Finished early? Here are some opportunities to explore what we have learned, and extra challenges requiring creativity and research.

In a classroom setting please divide the challenges between teams (this allows us to work through all the material in the time available).

.. only:: instructor
  
   .. admonition:: Instructor Notes 

      As usual the Explore section invites readers to reapply the material covered in a slightly different context or dataset.
 
      The use of selectors using the roads **type** attribute provides this opportunity.

.. admonition:: Explore Vendor Option Follow Line

   Vendor options can be used to enable some quite spectacular effects, while still providing a style that can be used by other applications.

   #. Update `line_example` with the following:

      .. code-block:: css

         * {
           stroke: ededff;
           stroke-width: 10;
           label: [level] " " [name];
           font-fill: black;
           -gt-label-follow-line: true;
         }

   #. The property **stroke-width** has been used to make our line thicker in order to provide a backdrop for our label. 

      .. code-block:: css
         :emphasize-lines: 3
      
         * {
           stroke: ededff;
           stroke-width: 10;
           label: [level] " " [name];
           font-fill: black;
           -gt-label-follow-line: true;
         }

   #. The **label** property combines combine several CQL expressions together for a longer label.

      .. code-block:: css
         :emphasize-lines: 4

         * {
           stroke: ededff;
           stroke-width: 10;
           label: [level] " " [name];
           font-fill: black;
           -gt-label-follow-line: true;
         }

      The combined **label** property::
         
         [level] " " [name]
         
      Is internally represented with the **Concatenate** function::

         [Concatenate(level,' #', name)] 

   #. The property **-gt-label-follow-line** provides the ability of have a label exactly follow a LineString character by character.

      .. code-block:: css
         :emphasize-lines: 6
      
         * {
           stroke: ededff;
           stroke-width: 10;
           label: [level] " " [name];
           font-fill: black;
           -gt-label-follow-line: true;
         }

   #. The result is a new appearance for our roads.

      .. image:: /style/img/line_label_4.png
   
.. admonition:: Challenge SLD Generation

   #. Generate the SLD for the following CSS.

      .. code-block:: css

          * {
            stroke: black;
          }

      What is unusual about the SLD code for this example?
   
   #. **Challenge:** Can you explain why this SLD still works as expected?

   .. only:: instructor
  
      .. admonition:: Instructor Notes       

         The generated SLD does not contain any stroke properties, even though black was specified::
  
            <sld:LineSymbolizer>
              <sld:Stroke/>
            </sld:LineSymbolizer>
  
         SLD considers black the default stroke color for a LineSymbolizer, so no further detail was required.

.. admonition:: Challenge Classification

   #. The roads **type** attribute provides classification information.
   
      You can **Layer Preview** to inspect features to determine available values for type.
   
   #. **Challenge:** Create a new style adjust road appearance based on **type**.

      .. image:: /style/img/line_type.png

      Hint: The available values are 'Major Highway','Secondary Highway','Road' and 'Unknown'.

   .. only:: instructor
      
      .. admonition:: Instructor Notes          

         Here is an example:
     
         .. code-block:: css
     
              [type = 'Major Highway' ] {
                  stroke: #000088;
                  stroke-width: 1.25;
              }
              [type = 'Secondary Highway' ]{
                  stroke: #8888AA;
                  stroke-width: 0.75;
              }
              [type = 'Road']{
                  stroke: #888888;
                  stroke-width: .75;
              }
              [type = 'Unknown' ]{
                  stroke: #888888;
                  stroke-width: 0.5;
              }
              * {
                 stroke: #AAAAAA;
                 stroke-opacity: 0.25;
                 stroke-width: 10;
              }

.. admonition:: Challenge SLD Z-Index Generation

   #. Review the SLD generated by the **z-index** example.
   
      .. code-block:: css

         * {
           stroke: black, #8080E6;
           stroke-width: 5px, 3px;
           z-index: 0, 1;
         }

   #. *Challenge:* There is an interesting trick in the generated SLD, can you explain how it works?

   .. only:: instructor
     
      .. admonition:: Instructor Notes    

         The Z-Order example produces multiple FeatureTypeSytle definitions, each acting like an "inner layer".
  
         Each FeatureTypeStyle is rendered into its own raster, and the results merged in order. The legend shown in the map preview also provides a hint, as the rule from each FeatureType style is shown.

.. admonition:: Challenge Label Shields

   #. The traditional presentation of roads in the US is the use of a shield symbol, with the road number marked on top.
   
      .. image:: /style/img/line_shield.png
   
   #. *Challenge:* Have a look at the documentation and reproduce this technique.

   .. only:: instructor
   
      .. admonition:: Instructor Notes      

         The use of a label shield is a vendor specific capability of the GeoServer rendering engine. The tricky part of this exercise is finding the documentation online ( i.e. :manual:`Styled Marks in CSS <community/css/styled-marks.html>`).
         
         .. code-block:: css
       
            * {
                stroke: black,lightgray;
                stroke-width: 3,2;
                label: [name];
                font-family: 'Ariel';
                font-size: 10;
                font-fill: black;
                shield: symbol(square);
            }
            :shield {
                fill: white;
                stroke: black;
                size: 18;
            }

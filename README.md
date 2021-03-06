# A technical guide on tier-one urban network for emergency response
The objective of this guide is to identify the most useful road network in a specific location to get to the main services around in case of emergency. This guide could be used in slums or other areas, regardless of size. However, areas at the scale of cities or even wards requires heavier computational analysis and a more complex process of decision making that is not covered in this guide.

This guide is based in the work of the department of Urbanization and Sustainable Cities[^1]  of GRADE and the lessons learned from the project Capacity Building for Smart Data and Inclusive Cities in India[^2]. 

## General and Software Requirements
This technical guide requires use of spatial information. So it needs software to process this kind of information. It’s highly recommendable to use QGIS[^3]. Download and install the latest LTR version. The algorithms referred on this guide come from QGIS. However, they have equivalents on similar software like ARCGIS since the processing tasks are public domain. 

Depending on the data availability it might be needed a location device like a smartphone. It’s highly recommendable to use a standalone device for more accuracy. The average accuracy of a smartphone range from 3 – 50 meters, depending on the location of the area of interest and the device’s technology. A dedicated device has a location accuracy of 0.5 – 1 meters. Using this kind of device will increase the precision of the location register and reduce the post processing time.

It’s is required to have access to the area of interest. A dedicated team to recognize the fieldwork and engage the local population will provide information that otherwise cannot be acquired. This team will be responsible to gather on the field spatial information and communicate it with the team in charge of spatial analysis.

Most of the communication should be based on thematic maps [(Figure 01)][1]. These maps are based on specific data regarding open spaces, markets, roads, physical risks, community services. Reports should gather imagery, location and commentaries. Is highly recommended to compose maps with applications like My Maps[^4]. Such applications allow you to easily build thematic maps based on the gathered information on the ground. It also makes it easier to share the information via link. 

Make sure that any photograph taken on field has also location data. It reduces human error at reporting location of any event. Consider the use of a single My Maps composition to stock each field work report organized into different layers.

Interoperability between this platform and qgis is reliable. To import/export the reports data just find the `Export to KML/KMZ` action and select the necessary information; a vector layer will be created. On the qgis environment, open the data. Although it will not present the same icons, color or style the data would be just the same. It will be organized by vector type; that is points, lines or polygons.

<img src="/figs/fig01.jpg" alt="Field Work Report" width="75%"/>

## Major tasks and workflow
There are 10 major task involved in this guide. Some of them are independent and some others are interdependent. [(Figure 02)][2] represents the most efficient interaction between them. In the graph, the task are organized so that it could produce an outcome as fast as possible. Use this workflow schema to take advantage of the independency among task and stay ahead in the project’s development. However, organizational or environmental factors might affect the flow of information. Use alternatives responding to the project’s environment.

<img src="/figs/fig02.jpg" alt="Suggested Work Flow" width="75%"/>

## Spatial scope definition
Defining the spatial scope means to decide the extension of the urban area that will be covered. This location, from now on called area of interest, should be easily identifiable. Use land administration information or toponym to identify the spatial shape of this place. A simple query [(Figure 3)][3] on any map online service like Google Maps[^5] , Bing[^6]  or Open Street Maps[^7]  will provide a quick idea of it.

<img src="/figs/fig03.jpg" alt="Boundaries of an AOI found on Google Maps" width="75%"/>

Identify socio spatial divisions inside the area of interest. That is non visible borders based on social features like economic status, familiarity, foundation or any other singularity. Make sure you have this data since it will be needed to address the allocation of some services. If this information is not easily available through any geospatial database (public or from any government agency) it must be consulted on field surveys.

## Terrain data acquisition
Acquire a digital elevation model for the area of interest. This might be acquired from the USGS Earth Explorer[^8] platform. Find the mission SRTM[^9] dataset, and find the most recent capture of the area of interest. Another characteristic to consider is resolution; a recommended one is 1 arc-second or 30m. Other sources of information might be the Copernicus Open Access Hub[^10].

The DEM will be the main source of information to derive morphological and hydrological analysis. As the DEM provided by any platform would cover a bigger size area than the area of interest, it is required to clip it to a more practical size [(Figure 4)][4] in order to make processing faster. Use the `Clip raster by extend` algorithm from the GDAL[^11] modules; find this and any other algorithm on the Processing Toolbox.

<img src="/figs/fig04.jpg" alt="DEM covering the area of Bhopal (Madhya Pradesh, India)" width="75%"/>

## Hydrological analysis
The hydrological analysis is composed by two calculus derived from the DEM. The Terrain Wetness Index and the Stream Power Index. The Terrain Wetness Index (TWI) provides a relative score showing the areas where the terrain is most probable to accumulate water. It’s defined as: 

```
TWI=ln⁡〖 UCA/tan⁡SR 〗
  
  Where:
  UCA = the local upslope catchment area
  SR =  local slope in radians
 ```

In the QGIS environment, apply the next sequence of algorithms to calculate it: Use the `Flow Accumulation` algorithm from the SAGA[^12] modules; apply it to the DEM clip. The result is the local upslope catchment area (UCA). The local slope is calculated with the native `Slope` algorithm; it should be transformed into radians. But first, the 0 values corrected since it would induce error when calculating natural logarithm; add 0.001 to the slope values.

Then, multiply the modified slope value 0.01745 times. In order to do this, use the `Raster Calculator` algorithm; use it for any raster algebra operation. Finally, calculate the log normal function. The UCA must be multiplied by the X and Y values of raster pixel dimensions.

The TWI [(Figure 5)][5] must be classified in 5 categories; quantile classification is suggested. Consider priority one or more classes of highest value. Use gathered information from field or imagery to make an informed and coherent prioritization.

<img src="/figs/fig05.gif" alt="DEM and TWI" width="75%"/>

The Stream Power Index (SPI) provides a relative score to potential flow erosion. It shows areas where fluids are more probable to have more velocity, hence endangering dwells. It’s defined as:

```
SPI=ln⁡〖(UCA*tan⁡SR)〗

  Where:
  UCA = the local upslope catchment area
  SR  = local slope in radians
```

In the QGIS environment, use both the UCA and SR previously calculated. Proceed to multiply them; also multiply times the X and Y values of raster pixel dimensions. As before, use the `Raster Calculator` algorithm.

The SPI [(Figure 6)][6] must be classified in 5 categories; quantile classification is suggested. Consider priority one or more classes of highest value. Use gathered information from field or imagery to make an informed and coherent prioritization.

<img src="/figs/fig06.gif" alt="DEM and SPI" width="75%"/>

Both the TWI and SPI will be used to identify the most compromised roads and places on the area of interest. If other sources of hydrological data are available, even better. Especially if those data are based on actual records of flooding or precipitation. Other calculus based on remote sensing imagery are also recommended; apply this if there is data available for before and after a referential flood event.

## Morphological analysis
The slope is a measure of inclination of the terrain. It’s also a derivate product of the DEM. The calculus provides an inclination value at any given area. It shows where inclination could be compromising dwells. In the QGIS environment (Figure 7), apply the GDAL algorithm `Slope` to calculate it. Most GIS software have a native algorithm for slopes since it’s a basic calculation. Consider that slope units might be expressed as degree, radians or even percent.

<img src="/figs/fig07.jpg" alt="Slope algorithm's location and parameters" width="75%"/>

Use a benchmark value[^13] to identify areas where there could be any risk related to slope. That value should be used to identify areas where the slope value is below or above the value, ergo at risk or not. The areas above the benchmark value should be prioritized. Use gathered information from field or imagery to make an informed and coherent decision. Also consider that the units from your slope calculation correlates to the benchmark value.

## Urban and Land Administration Data acquisition
In order to analyze spatial data it is necessary to gather as much geo-referenced information as possible. This project requires four types of data at least. Administrative boundaries like wards, slums or other known areas should be collected from government agencies like the Department of Science & Technology; it’s common that District Geo-portals [(Figure 8)][8] allow to visualize spatial information. Make a formal request of information, get an alternative source of information or even scrap it.

<img src="/figs/fig08.jpg" alt="Online Platform of the Bhopal District Geoportal" width="75%"/>

The other three types of information are road network, water bodies and services. Which could be gathered quickly from open sources like Geofabrik[^14]. Once the data has been collected, the gaps of information should be identified and completed. Particularly for urban network information.

Complete the gaps with data from multiple sources like remote sensing imagery, spatial surveys or even government agencies. Depending on the case, it might be necessary to buy the data from a third party organization like Image Hunter[^15]. Producing the information might be a choice if there is a drone survey provider on the area. In both cases, the imagery will be used as a reference to add the new road information manually. That is, drawing each road as precisely as possible.

However, a field survey dedicated to road network data might cover the gaps of information and other aspects. This is done by walking around the area with a tracking device. Using the GPS from a smartphone is possible but is highly recommended to use a standalone GPS device for accuracy.

Once the information has been produced [(Figure 9)][9], combine all those layers into a single one. Then, prepare the improved urban network for spatial analysis. That is, process it so that is has the next characteristic:

-	A line feature for each piece of road between corner to corner of the streets. 
-	A line feature represents a road regardless if road is bidirectional or not.

<img src="/figs/fig09.jpg" alt="Road layer stacked by different sources" width="75%"/>

## Field work
Make exploratory walks to recognize major features in the area like roads, open areas, services of any kind, location of local leaders, local government agencies, water bodies or any other natural feature. Acquire on the field information of the most important uses. Identify them by talking to community members and leaders. Arrange a meeting if necessary.

Use categories to classify the data [(Figure 10)][10]. The following might be useful for any situation: health services, groceries or food providers, waste collection, community public services, places of worship or temples, and parks or open spaces.

<img src="/figs/fig10.jpg" alt="Services and Places location classified by type" width="75%"/>

Identify the priority places among all of them. Focus groups or participatory techniques will help you identify the hierarchy in each category; which must be based on use intensity.

## Compromised Roads Identification
Use the updated Road Network to juxtapose the hydrological and morphological analysis [(Figure 11)][11]. Make a visual observation to identify intersections and evaluate how much length of roads are compromised to represent a real risk. Once the compromise roads are identified, these observations will be used to improve the tier one network. 

<img src="/figs/fig11.jpg" alt="Updated Network and Hydrological Analysis" width="75%"/>

This is mandatory since otherwise the analysis would produce inaccurate results. Remember to get in contact with the government agency that manages this information; it might reduce the amount of efforts on this task.

## Shortest Distance Network to Services
For every class of service or main use that has been previously prioritized, you should calculate a shortest distance network. A shortest distance network connects a pair of data points with another set of points in the most efficient way. One data set must be a priority class service. It means that this calculus must be repeated for each of class. The other data set will the same for all calculus.

This data set should contain the points with the lowest score in the network’s hierarchy. Those points are connected to the network by one road. This score is called degree of centrality[^16], which is defined as the number of links incident upon a node. In the QGIS environment [(Figure 12)][12], apply the `v.net.centrality` algorithm; which is a GRASS[^17] module used to calculate the degree of centrality (also known as Cd) and other centrality measurements.

<img src="/figs/fig12.jpg" alt="Location and parameter of 'v.net.centrality' algorithm" width="75%"/>

Once the degree of centrality is calculated, the ones with lowest score must be selected. This is achieved by using the `Select Features` action. Write the minimum value available from the degree of centrality column. Depending on the number of vertex of the network, the values would be different from the example [(Figure 13)][13]. However, if the network was cleaned as described before, it should have no more than 5 different values; each one for each degree.

<img src="/figs/fig13.jpg" alt="Degree of Centrality for a given urban network" width="75%"/>

To calculate the shortest distance path use the `v.net.distance` algorithm, it’s another module from the GRASS library. It requires three data inputs, which corresponds to the starting point of the shortest path, the end points, and the network that contains both datasets. Use as `Input Vector from Points Layers` the previously calculated nodes with minimum Cd, use as `Input Vector to Layer` the prioritize nodes from each one of the use categories, and as the `Input Vector Line Layer` the updated version of the network.

Once each calculus is done, clean each result by eliminating duplicity of features in each network. Use the `Extract by Location` native algorithm to select any feature from the updated network with each result from the `v.net.distance` calculus. This algorithm will produce a new layer without any duplicated feature. Make sure that the `Geometric Predicate` for all cases is `Overlap`; otherwise other roads will be added or discarded.

Then all the networks should be combined into one single network containing all shortest distance networks. Use the `Merge vector layer` algorithm from the Processing Toolbox. Then, as in the previous networks, eliminating duplicates is necessary. Use the same procedure as before.

The combined network is a subset of the main one [(Figure 14)][14]. It contains all shortest path to any service from the weakest points in the main network. However is not optimized yet, as it present redundancies among services from different classes that are near. Therefore, a tier one network should be calculated.

<img src="/figs/fig14.jpg" alt="Combined Shortest Distance Network on top of the original Network" width="75%"/>

## Steiner Tree
A Steiner tree[^18] is an optimization solution for networks. It identifies the most efficient way to communicate a subset of nodes inside a network. This algorithm will be used to connect the nodes with the highest score in the network’s hierarchy. But this time, the centrality measure must be the betweenness centrality[^19] (also known as Cb). Its calculus follows the same `v.net.centrality` algorithm as before. To identify the nodes with the highest values, use the `Statistics` Panel and find the lower end of the 4th Quantile [(Figure 15)][15].

<img src="/figs/fig15.jpg" alt="Value Identification and Selection of Q4 Betweenness Centrality" width="75%"/>

Those nodes are the most useful among the network; those are needed frequently to get the shortest paths to one node to another. Apply the calculus to the previous network; not to the whole dataset since it would be inaccurate. As in a previous step, the result must be filtered. Use the `Select by Feature` action to identify the nodes with a greater or equal value to the 4th Quantile.

Those nodes will be used to calculate its Steiner Tree. In the qgis environment, apply the `v.net.steiner` algorithms to calculate it. Despite the apparent complexity of this module, the only inputs that are required are the `Input Vector Line Layer` and the `Center Point Layer`. Those inputs are the previously obtained network and its nodes with the highest value of centrality Cb. It isn’t required to add any other piece of information. 

The result should be another network subset [(Figure 16)][16]. This dataset contains few features. It also contains the most valuable roads in term of connectivity. In the next process, this results will be complemented with other roads.

<img src="/figs/fig16.jpg" alt="Steiner Tree network made from nodes with Cb value >= Q4" width="75%"/>

# Optimal Network 1.0
The Optimal Network 1.0 uses the previously calculated Steiner Tree as the main input. The Steiner Tree must be complemented with roads that enhance its connectivity to the outer network of the area of interest [(Figure 17)][17]. Use the combined network to services to pick the roads that best fit this requirement. Use the knowledge gathered from field work and imagery to decide properly.

<img src="/figs/fig17.jpg" alt="An Optimal Network made from the Steiner tree" width="75%"/>

Now use the Compromised Roads data to evaluate for alternatives among the current roads of the network. Select alternatives in case extended areas of roads are compromised and whenever is possible.

Evaluate the result by identifying the continuity among roads. A prioritized urban network presents straightness. Consider that turning roads is expensive in term of energy and time consumption. Whenever possible, consider this criteria to make small changes.

## Community Consultation
Present this result to the community and local leaders. Make sure the information is clear for them [(Figure 18)][18]. Consider the use of one or even more maps to communicate the process of prioritization. Verify a proper use of colors and visual aids like icons. Avoid misleading communication. If this issue is not addressed properly before meetings, it will not be easily solved while debating.

<img src="/figs/fig18.jpg" alt="A clear map with the information needed and visual references" width="75%"/>

Make sure none of the visual content is somehow offensive. Colors often have different interpretations across cultures. Same as icons; representation of temples is not the same for Christians, Jews or Islamic. Also verify the use of proper language. 

Collect the observations from the discussion. Make sure you have spatial observation. Comments from the community must have spatial information and value judgment. Use participatory tools to make everyone involved as much as possible.

# Optimal Network 2.0
Make the adjustments according to the commentaries from the meetings [(Figure 19)]. Make further consultation with other stakeholders and experts. Present new results and validate them. It is important to address the commentaries from the community and give proper explanation of how their ideas where considered; even if it means to explain why a suggestions wasn’t taken.

<img src="/figs/fig19.jpg" alt="The final version of the Optimal Network" width="75%"/>

## Area of Coverage Identification
A risk management application for area of coverage identification is micro-zoning [(Figure 20)][20]. Which is based on network classification by proximity to a given service. The `v.net.alloc` algorithm is a GRASS module that classifies a network into subsets corresponding to the closest feature of a give use or service. Which might be used to delimitate micro-zones for temporary shelters, field hospital or food provision. The parameters are an Input Vector Line and Center Point layers.

<img src="/figs/fig20.jpg" alt="Services Area of Coverage" width="75%"/>

parameters are an Input Vector Line and Center Point layers.
The results could be processed to derive a polygon for each zone. Since there is not an algorithm for this task, it needs to be done manually. However, a convex hull[^20] is easily computed with the native `Convex Hull` algorithm. Use it as a basis for the polygons or do it from scratch. In any case, fine tune the micro-zones with on the field information.

# Behavioral Study by Place
Once the location of main services is done, further questions about behavior around its location could be addressed. By surveying, observing or counting on community’s knowledge, some of these questions could be addressed. One of the questions is agglomeration [(Figure 21)][21]. Use intensity beyond safety is identifiable by any of the previous methods. Gather the information and mark the most crowded places. Add more complexity to the selection process if needed. 

<img src="/figs/fig21.jpg" alt="Case Study of Agglomeration" width="75%"/>

Use attributes to add scores, a rank values or a Boolean factor. This might be helpful to filter the points that meet the requirements. The same procedure could be applied to other features like roads, slums or even wards.

This process is also applicable to identify wards based on vulnerability. If census information is available, it could be used to build an index based on the relevant variables for a given case (i.e. water supply, drainage service, household condition, etc.). Use the `Field Calculator` to add variables and summarize them.

## Service Distribution
Waste management is an application of service distribution [(Figure 22)][22]. It requires the identification of suitable places for waste collections. Use other compatible uses as a proxy of suitability; also considering wind direction as the garbage smells travels and transports unhealthy bacteria. Other criteria is accessibility, defined as useful road infrastructure for waste collector trucks. Use remote sensing imagery or field reports to address the relevant issues. 

<img src="/figs/fig22.jpg" alt="Waste Management proposal" width="75%"/>

If there isn’t any other referential information to distribute services it is possible to segment a network based on their geometrical features. The `k-means clustering` native algorithm divides any vector layer into a given set of clusters. Then use the `mean coordinate` algorithm to identify the center of each part. Although it’s not based on use or any other feature rather than spatial distance, it should provide a starting point to discuss distribution. Furthermore, the center points of each zone might be used to evaluate any proposed spot.

[1]: figs/fig01.jpg
[2]: figs/fig02.jpg
[3]: figs/fig03.jpg
[4]: figs/fig04.jpg
[5]: figs/fig05.gif
[6]: figs/fig06.gif
[7]: figs/fig07.jpg
[8]: figs/fig08.jpg
[9]: figs/fig09.jpg
[10]: figs/fig10.jpg
[11]: figs/fig11.jpg
[12]: figs/fig12.jpg
[13]: figs/fig13.jpg
[14]: figs/fig14.jpg
[15]: figs/fig15.jpg
[16]: figs/fig16.jpg
[17]: figs/fig17.jpg
[18]: figs/fig18.jpg
[19]: figs/fig19.jpg
[20]: figs/fig20.jpg
[21]: figs/fig21.jpg
[22]: figs/fig22.jpg

[^1]: Head by R. Fort and A. Espinoza. For more into referred work, consult [here](https://www.grade.org.pe/areas-de-investigacion/urbanizacion-y-ciudades-sostenibles/)
[^2]: A co-authored project with the Institute for Development Studies (UK) featuring [Samarthan Centre for Development Support](https://www.ids.ac.uk/projects/capacity-building-for-smart-data-and-inclusive-cities-sdic/) (India)
[^3]: QGIS is a free and open-source cross-platform desktop geographic information system application that supports viewing, editing, and analysis if geospatial data.
[^4]: My Maps is an online service run by Google. It allows users to build personalized maps for their own use or to share.
[^5]: Google Maps is an online service run by Google. Allows users to consult and share geospatial information.
[^6]: Bing is an online service run by Microsoft. Allows users to consult and share geospatial information.
[^7]: Open Street Maps is a collaborative project to create a free editable geographic database of the world.
[^8]: USGS Earth Explorer is a client/server interfaces that provides access to the US Geological Survey Earth Resources Observation and Science Center archive.
[^9]: The Shuttle Radar Topography Mission is an international research effort to obtain digital elevation models on a near global scale. 
[^10]: The Copernicus Open Access Hub (previously known as Sentinels Scientific Data Hub) provides complete, free and open access to Sentinel-1, Sentinel-2, Sentinel-3 and Sentinel-5P user products.
[^11]: GDAL is an Open Source library for raster and vector geospatial data formats that is released by the Open Source Geospatial Foundation.
[^12]: SAGA GIS is a geographic information system (GIS) computer program, used to edit spatial data.
[^13]: In the city of Lima, 24% is the slope value where urban areas start showing signs of restraints. Service and mobility access decreases as physical risk increases. At 43% is by any means recommend to inhabit.
[^14]: Geofabrik is a provider of geodata, its database comes mainly from Open Street Map and it’s also open data.
[^15]: Hunter is an on-demand provider of geodata; it has multiple sources of remote sensing data with high resolution. Check the WorldView 2 dataset; it’s one of the most high resolution datasets available.
[^16]: Degree of centrality (Cd) is one of many measurements of centralities for network analysis and graph theory. It is also the simplest one. It can be interpreted as the immediate possibility of transmission of information through the vertex of a network.
[^17]: Geographic Resources Analysis Support System (GRASS GIS) is a GIS software suite used for geospatial tasks including analysis. QGIS provides a user-friendly interface for GRASS modules (units of software that processes data).
[^18]: A Steiner tree is an optimization solution for a given subset of vertex on a network. It’s the network composed by the shortest paths between the given subset of vertex.
[^19]: Betweenness centrality (Cb) is a measure of centrality based on shortest distance among vertex. Which are scored according to the number of times they are part of the shortest path between each pair of vertex on a given network.
[^20]: In geometry, the convex hull of a shape is the smallest convex set that contains it. It may be defined either as the intersection of all convex sets containing a given subset of a Euclidean space.


# Election in France GIS data

The idea of this session is to grab data of election in France.

Since June 2023, INSEE has put online the addresses depending on the polling stations.


## Data to download

### INSEE REU (Répertoire électoral unique / Single electoral register)

Correspondence files between polling stations and the addresses of voters depending on them allow the construction of polling station areas. These facilitate the comparison of electoral data with the socio-demographic characteristics of voters attached to these areas. These files are drawn up from the Single Electoral Register (REU), established by law no. 2016-1048 of August 1, 2016 renewing the procedures for registration on the electoral lists, and which was implemented from January 1 2019.

See at INSEE description at https://blog.insee.fr/a-vote-a-chaque-bureau-de-vote-ses-electeurs/

Download sources : see page here https://www.data.gouv.fr/fr/datasets/bureaux-de-vote-et-adresses-de-leurs-electeurs/
(recommanded format : csv file) : https://www.data.gouv.fr/fr/datasets/r/5142e8a9-15f3-4216-b865-deeeb02dde70


### BAN : Base Adresses Nationales 

See at : https://adresse.data.gouv.fr/
For our work, it will not be necessary

All addresses in France : 
by departement : https://adresse.data.gouv.fr/data/ban/adresses/latest/csv
All France : https://adresse.data.gouv.fr/data/ban/adresses/latest/csv/adresses-france.csv.gz

## Data operation

### 1. Load Reu Data

File is reu_adresses_france_31.gpkg
Around 309k points.

Classify / Semiology - categorized with variable id_brut_bv_reu

But we have points, maybe it can be usufull to have polygons.


### 2. Build Voronoi polygons from Reu Points

See https://github.com/InseeFrLab/traitement-adresses-REU for a better methodology, but more time consuming.

The partitioning of a plane with n points into convex polygons such that each polygon contains exactly one generating point and every point in a given polygon is closer to its generating point than to any other. A Voronoi diagram is sometimes also known as a Dirichlet tessellation. The cells are called Dirichlet regions, Thiessen polytopes, or Voronoi polygons.

![a66b373a9d955d1e75e0c2ff6c32c5dc.png](:/78f7abbfa5cb4396bc751d8e1812a275)

![9f77d5564d5754c6f8769f2ef991276b.png](:/39abfd73e76045e9a7f895f1af608754)

In Menu Vector / Geometry tools / Voronoi Polygons

Same attributes are export in a new polygon layer.

But Voronoi create a too large bounding.

Need to be cut...

### 3. Load Administrative Data, municipalities of Haute-Garonne

589 municipalities

No secret but....

Treatments are needed : 

1. aggregate 589 municipalities in one unique polygon with Menu vector / Geoprocessing tools / Dissolve
2. Obtain a buffer (1km) on this new polygon.
3. Intersection voronoi with dissolved and buffered layer of municipalities : Menu Vector / Geoprocessing tool / Intersection : 
- first parameter : voronoi uncut
- second parameter : commune 31 dissolved and buffered
4. Classify / Semiology - categorized with variable id_brut_bv_reu
5. It can be useful to dissolve this new layer by id_brut_bv_reu in order to reduce the number of poygons and for future visualisation.
So : Menu vector / Geoprocessing tools / Dissolve, choose attribute id_brut_bv_reu

### 4. Analysis that can be possible

1. Number of points by areas
2. Density of points by areas
3. Size and shape of areas : standard polygons / fragmentation of shapes (multipolygons); etc.

### 5. Electoral data

Premier tour :
https://www.data.gouv.fr/fr/datasets/election-presidentielle-des-10-et-24-avril-2022-resultats-definitifs-du-1er-tour/

Data all France by REU :
https://www.data.gouv.fr/fr/datasets/r/79b5cac4-4957-486b-bbda-322d80868224

0. whose the winner :
- create en field score_max with :
max(
to_real(replace( "% Voix/Exp", ',', '.')),
to_real(replace( "field_34" , ',', '.')),
to_real(replace( "field_41", ',', '.')),
to_real(replace( "field_48" , ',', '.')),
to_real(replace( "field_55", ',', '.')),
to_real(replace( "field_62" , ',', '.')),
to_real(replace( "field_69", ',', '.')),
to_real(replace( "field_76" , ',', '.')),
to_real(replace( "field_83", ',', '.')),
to_real(replace( "field_90", ',', '.')),
to_real(replace( "field_97" , ',', '.')),
to_real(replace( "field_104", ',', '.'))
)

- create a field winner (winner name) with :
CASE
WHEN to_real(replace( "% Voix/Exp", ',', '.')) = "score_max" THEN "Nom" 
WHEN to_real(replace( "field_34" , ',', '.'))  = "score_max" THEN  "field_31"
WHEN to_real(replace( "field_41", ',', '.')) = "score_max" THEN  "field_38" 
WHEN to_real(replace( "field_48" , ',', '.')) = "score_max" THEN  "field_45" 
WHEN to_real(replace( "field_55", ',', '.')) = "score_max" THEN  "field_52" 
WHEN to_real(replace( "field_62" , ',', '.')) = "score_max" THEN  "field_59" 
WHEN to_real(replace( "field_69", ',', '.'))  = "score_max" THEN  "field_66" 
WHEN to_real(replace( "field_76" , ',', '.')) = "score_max" THEN  "field_73" 
WHEN to_real(replace( "field_83", ',', '.')) = "score_max" THEN  "field_80" 
WHEN to_real(replace( "field_90", ',', '.'))  = "score_max" THEN  "field_87" 
WHEN to_real(replace( "field_97" , ',', '.')) = "score_max" THEN  "field_94" 
WHEN to_real(replace( "field_104", ',', '.'))  ="score_max" THEN "field_101" 
END

1. Prepare the joint with an index : codecommune_codebv

- create a field bv_nozero with  ltrim( "Code du b.vote", '0') !!! type must be string !!!
- create a field code_com_zero with  substr( '000' || "Code de la commune", -3)
- create code_com_bv (string, size=10) with  "Code du département"  ||  "code_com_zero"  || '_' ||  "bv_nozero" 

2. join in layer voronoi_intersection_with_communes31_dissolved_by_bv_reu with the layer results_election_2022_t1 — resultatsparniveauburvott1franceentiere on attributes 

## Analysis

### Explore Data

1. Classify or pie chart
dont forget to use replace("val", ',', '.') in pie or diagrams

2. Compute the shortest lines to center :
- create a new scratcg layer with one points
- in processing, compute shortest line between (first parameter is the polygon layer, second parameter is the point layer)

3. Data analysis with dataplotly
Install plotly in extensions
try 2D histo with distance / winner
try correlation distance between field melanchon or field lepen

4. try with now 2 points in scratch layer
Préfecture / Sous-préfecture (Saint-Gaudens)

5. new plots etc.

### Add Filosofi data

Production of a set of indicators on declared income (before redistribution) on the one hand, and on available income (after redistribution and imputation of undeclared financial income) on the other hand, at the municipal level, supra- municipal and sub-municipal:
- usual indicators for analyzing income distribution (numbers, quartiles, deciles, median, etc. of income per consumption unit) over the entire population as well as over sub-populations
- monetary poverty rate
- income structure indicators

https://www.insee.fr/fr/metadonnees/source/serie/s1172

For other data, look at https://www.insee.fr/fr/statistiques?debut=0&theme=81&geo=IRIS-1 (data by iris)

Available in in dir data/filosofi

Variables dictionnary for this file
https://www.insee.fr/fr/statistiques/6215138?sommaire=6215217#dictionnaire

### Add Iris with structure of population (2017)

Coproduction between :

IGN : https://geoservices.ign.fr/contoursiris

and INSEE : 

Data : https://www.insee.fr/fr/statistiques/7632446?sommaire=7632456

Dashboard : https://www.insee.fr/fr/outil-interactif/5367857/tableau/20_DEM/21_POP#:~:text=Au%201%E1%B5%89%CA%B3%20janvier%202023%2C%20la,millions%20contre%2032%2C9%20millions.

### Correlation

In Menu Plugins / Manage plugins add Processing R Provider


### Kepler.gl

Export data to Geojson

Deposit into https://kepler.gl/demo


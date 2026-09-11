# GeMS-Stratigraphic-Column-Generator
This script generates stratigraphic columns from a USGS geological unit shapefile and a Digital Elevation Model.

1. Purpose
This document describes the methodology used to generate the schematic stratigraphic column figures compiled in the “Geological Atlas of Washington State” book, and evaluates that methodology against standard practice in structural geology and stratigraphy.  It is intended as a methods reference for anyone using, reviewing, or citing those figures.
2. Inputs
•	A GeMS-style (Geologic Map Schema) geologic units polygon shapefile, carrying at minimum a unit code field (MapUnit) and, where available, a numeric age field (Ma), a period/age-name field, and/or the GeMS AreaFillRGB color attribute.  For the book, the USGS ‘Geology shapefiles for the United States and Australia’ was used (McCafferty et al. 2023).
•	A digital elevation model (DEM) raster covering the same area, in any common raster format readable by rasterio (e.g., GeoTIFF).
3. Processing Steps
3.1 Color assignment
Each unit is assigned a display color using, in priority order: (a) its own GeMS AreaFillRGB value, if present and parseable; (b) a standard color looked up by matching the unit's geologic period against a fixed CGMW/USGS geologic-time-scale palette (the same family of colors used in FGDC-STD-013 map symbolization); or (c) a neutral gray fallback if neither is available. Matching is tolerant of common real-world variations — full period names, epoch names, legacy “Tertiary” usage, and standard one/two-letter USGS age-symbol codes (e.g., “Q”, “K”, “Tr”) are all recognized.
Because a single unit is frequently mapped as many separate polygon patches (especially Quaternary surficial units), attribute values are pulled by scanning all of a unit's patches for the first non-blank value in each field, rather than relying on whichever patch happens to be first in the table — the latter is the default behavior of common GIS dissolve operations and was found during development to silently assign incorrect (default gray) colors to units whose first-listed patch had incomplete attribution.
3.2 Stratigraphic ordering
Units are ordered oldest-to-youngest (bottom to top of the column) using, in order of preference: an explicit stratigraphic-order field (GeMS HierarchyKey convention, where smaller values are younger); a numeric age field, if HierarchyKey is unavailable; or, as a last resort, mean elevation within the unit's outcrop footprint, with a printed warning, since elevation-based ordering is a weak proxy that can be wrong wherever younger material does not simply overlie older material at higher elevation (e.g., inverted topography, faulted or overturned sections).
3.3 Thickness calculation — “apparent thickness”
For each unit, the DEM is sampled within the unit's mapped outcrop footprint, and the unit's displayed thickness is calculated as:
apparent thickness = (maximum surface elevation − minimum surface elevation) within the unit's mapped outcrop
This value is deliberately and consistently labeled “apparent thickness” throughout the figures, scripts, and output tables — see Section 5 for why that distinction matters.
4. Known Limitations and Data-Quality Notes
•	Apparent thickness is not corrected for structural dip (see Section 5) — it is a schematic, relative value, not a measured or true stratigraphic thickness.
•	DEM-relief-based thickness assumes the full vertical extent of a unit is exposed within its mapped footprint; units truncated by cover, faulting, or incomplete exposure will read as thinner than they actually are.
•	Elevation-based unit ordering (used only when no order or age field is available) is a weak fallback and should be checked against known geology.
5. Assessment Against Geologic Community Standards
Structural geology and stratigraphy have a long-standing, well-established, and unambiguous distinction between two quantities:
•	True (or stratigraphic) thickness — the perpendicular distance between a unit's upper and lower contact. This is the scientifically meaningful measurement of how thick a rock unit actually is.
•	Apparent thickness — the distance between the same contacts measured in any other direction, including straight up-and-down (elevation difference).  Apparent thickness equals true thickness only in the special cases of perfectly horizontal beds measured vertically, or perfectly vertical beds measured horizontally; in every other case it overstates the true thickness, more severely as dip increases.
This terminology is consistent across structural geology textbooks and course materials (e.g., Marshak & Mitra's Basic Methods of Structural Geology; university structural geology lab manuals), petroleum and wellbore-geosteering literature (which relies on the same true/apparent distinction for true stratigraphic thickness, or TST), and recent digital-outcrop and GIS-automation research.
Given that standard, unambiguous distinction, the elevation-relief method used here is best characterized as follows:
•	The terminology is accurate: labeling the output “apparent thickness” and explicitly disclaiming it as a substitute for a measured section or cross-section is the correct and expected framing under standard usage. A geologist reviewing the scripts or figures on those terms would find the labeling appropriate and not misleading.
•	The method itself is a simplification recognized in the literature (e.g., in digital outcrop correlation tools) as valid only for flat-lying strata measured on a vertical exposure face; it does not perform the dip correction (true thickness = apparent thickness × a trigonometric function of dip, or equivalent outcrop-width-and-dip formulas) that is standard practice for reporting true stratigraphic thickness from map data.
•	It would not be considered accurate or acceptable by the geologist community if presented as true stratigraphic thickness, in areas of significant structural dip, folding, or faulting, or as a substitute for a measured section, drill core, or a properly constructed cross-section perpendicular to strike.
•	It is a defensible schematic/relative visualization — useful for showing stratigraphic order, approximate relative unit thickness, and general lithology/age at a glance — provided it continues to be clearly labeled as apparent/schematic and not represented as a measured or true-thickness stratigraphic column, which is exactly how the underlying scripts already document it.
6. Recommendations if Greater Rigor Is Needed
1.	Incorporate bedding attitude (strike/dip) where available — e.g., from a GeMS OrientationPoints table — and apply a standard dip correction (true thickness = apparent thickness × cos(dip), for the simple vertical-relief case, or the fuller outcrop-width formula where appropriate) rather than reporting raw elevation relief.
2.	Where dip data is unavailable across an entire unit, consider automated orientation-extraction methods now used in the GIS literature (moving-window best-fit-plane algorithms applied to digitized contact traces) rather than a uniform assumed dip.
3.	For any unit or area where the column will inform a decision (rather than serve as an overview graphic), replace the DEM-derived value with a real measured section, borehole log, or a cross-section constructed perpendicular to strike.
References
FGDC. Digital Cartographic Standard for Geologic Map Symbolization (FGDC-STD-013-2006) — basis for the period-color and lithology-symbolization conventions used in Section 3.1. https://ngmdb.usgs.gov/fgdc_gds/geolsymstd/
Fossen, H. Structural Geology (Cambridge) — “Thickness and depth” chapter, formal definitions of thickness, apparent thickness, and outcrop width. https://www.cambridge.org/core/books/abs/structural-geology/thickness-and-depth/EF5931D071BCCAA367F3BD98455B3DCD
Kremer, K. et al. “InCorr: Interactive Data-Driven Correlation Panels for Digital Outcrop Analysis” — explicitly discusses the elevation-difference simplification for thickness and its limits. https://arxiv.org/pdf/2007.11512
Marshak, S. & Mitra, G. Basic Methods of Structural Geology — standard reference for true vs. apparent thickness calculation from strike lines and dip. https://www.geo.utexas.edu/courses/420k/PDF_files/LABS/gm2lab.pdf
McCafferty, A.E., San Juan, C.A., Lawley, C.J.M., Graham, G.E., Gadd, M.G., Huston, D.L., Kelley, K.D., Paradis, S., Peter, J.M., and Czarnota, K., 2023, National-scale geophysical, geologic, and mineral resource data and grids for the United States, Canada, and Australia: Data in support of the tri-national Critical Minerals Mapping Initiative (ver 1.1, March 2025): U.S. Geological Survey data release, https://doi.org/10.5066/P970GDD5.
Open Educational Alberta. Overview of Geological Structures Part 1: Strike, Dip, and Structural Cross-Sections — worked example of apparent vs. true bed thickness. https://www.saskoer.ca/geolmanual/chapter/overview-of-strike-dip-and-structural-cross-sections/

<img width="468" height="634" alt="image" src="https://github.com/user-attachments/assets/24465255-d1f6-4e86-a86c-72b9643894bb" />

<img width="139" height="612" alt="image" src="https://github.com/user-attachments/assets/d214c1ed-82dc-48e9-afc7-92cdb6ca2ddb" />


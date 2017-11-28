# Aggregating collections from different institutions

Demo integrating the collections from World Digital Library (which contains collections from Library of Congress, British Library, among others), University of Washington and University of British Columbia, using IIIF and Mirador viewer.

## [WWI & WWII Posters](https://open.library.ubc.ca/collections/wwposters)

This collection contains fifty nine posters, broadsides, and ephemera from World War I and II, published in Canada, Belgium, England, France, Germany, and the United States. There are similar collections across the web also using IIIF that could be aggregated in a web application, such as:

- World Digital Library War Posters
https://www.wdl.org/en/search/?additional_subjects=War%20posters

- University of Washington War Poster Collection
https://content.lib.washington.edu/postersweb/index.html
http://digitalcollections.lib.washington.edu/cdm/search/collection/posters

![mirador_WWposters_v02.gif](https://github.com/carolamigo/ubc_mirador_WWposters/blob/master/mirador_WWposters_v02.gif)

### How to do it:

- Install Mirador viewer following the instructions here: https://github.com/ProjectMirador/mirador

- Go to the root folder of the Mirador installation in your machine and open the file index.html (https://github.com/ProjectMirador/mirador/blob/develop/index.html). We need to replace the manifests of this template file with the manifests of our collections.

- To get manifests for UBC collection, download collection metadata using the Open Collections Research API. A php script to batch download is provided at OC API Documentation page > Download Collection Data. This script returns a folder containing one RDF file per collection item (or XML, JSON, any format preferred). We are going to use N-triples because the file is cleaner (no headers or footers), what makes the merging easier later. Edit the script following the instructions on the documentation page and run it using the command:

`$ php collection_downloader.php --cid wwposters --fmt ntriples`

- Merge the files using the Unix cat command:

`$ cat * > merged_filename`

- Convert merged file obtained to a tabular format. Import project in Open Refine using the RDF/N3 files option. No character encoding selection is needed.

- Build IIIF manifests. Create a new column named “manifest” based on the column “http://www.europeana.eu/schemas/edm/isShownAt”, using the expression:

`"https://iiif.library.ubc.ca/presentation/cdm.wwposters."+value.substring(10,19).replace(".","-")+".0000" + "/manifest"`

- Create the code line ready to be inserted in the index.html file by creating a new column named “manifest_code” based on the “manifest” column, using the expression:

`"{ \"manifestUri\": \""+value+"\", \"location\": \"University of British Columbia\"},"`

- Export the “manifest_code” column by clicking on Export > Custom tabular exporter. Select only the “manifest_code” column, do not output column headers or empty rows. On the download tab, select “Excel” and click download. Open the excel file, copy the column, and paste it in the appropriate position in the original index.html file, replacing the old code. 

- To get manifests from the University of Washington Collection, get page source code from the link http://digitalcollections.lib.washington.edu/cdm/search/collection/posters, and import as line based data in OpenRefine.

- Create a new column named “item_id” using the expression below to extract item IDs:

`value.match(/.*item_id=\"(\d\d).*/).join("")`

- Trim leading and trailing whitespaces of the “item_id” column by selecting “Edit cells” > “Common transforms” > “Trim leading and trailing whitespaces”.

- Build the manifest code by creating a new column named “manifest_code” based on the “item_id” column using the following expression:

`"{ \"manifestUri\": \"http://digitalcollections.lib.washington.edu/digital/iiif-info/posters/"+value+"\", \"location\": \"University of Washington Libraries\"},"`

- Export the “manifest_code” column by clicking on Export > Custom tabular exporter. Select only the “manifest_code” column, do not output column headers or empty rows. On the download tab, select “Excel” and click download. Open the excel file, copy the column, and paste it in the appropriate position in the original index.html file, below UBC manifests. 

- To get item ids from the Library of Congress collection, use the following microdata extractor http://microdata-extractor.improbable.org/ on the following URL: https://www.wdl.org/en/search/?additional_subjects=War%20posters. Import the resulting JSON file to Open Refine.

- Create a new column names “item_id” based on the column “_ - properties - url - url”, using the following expression:

`value.replace(/[^\d]/, " ")`

- Trim leading and trailing whitespaces of the “item_id” column by selecting “Edit cells” > “Common transforms” > “Trim leading and trailing whitespaces”.

- Build the manifest code by creating a new column named “manifest_code” based on the “item_id” column using the following expression:

`"{ \"manifestUri\": \""+"https://www.wdl.org/en/item/"+value+"/manifest"+"\", \"location\": \"World Digital Library\"},"`

- Export the “manifest_code” column by clicking on Export > Custom tabular exporter. Select only the “manifest_code” column, do not output column headers or empty rows. On the download tab, select “Excel” and click download. Open the excel file, remove the comma from the end of the last line, copy the column, and paste it in the appropriate position in the original index.html file, below University of Washington manifests. 

- Open the [index.html](https://github.com/carolamigo/ubc_mirador_WWposters/blob/master/index.html) file on your browser to see the final result. Click on Add item to see the menu with options.

